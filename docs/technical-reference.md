---
layout: page
title: MarketDAO Technical Reference
permalink: /docs/technical-reference/
---

## Table of Contents

1. [Contract Architecture](#contract-architecture)
2. [Security Features](#security-features)
   - [Governance Token Vesting](#governance-token-vesting)
   - [Lazy Vote Token Distribution](#lazy-vote-token-distribution)
   - [Purchase Restrictions (Optional)](#purchase-restrictions-optional)
   - [Join Request System](#join-request-system)
   - [Snapshot-Based Voting Power](#snapshot-based-voting-power)
3. [Security & Scalability](#security--scalability)
4. [Known Limitations & Design Decisions](#known-limitations--design-decisions)
5. [MarketDAO Contract](#marketdao-contract)
6. [Proposal Base Contract](#proposal-base-contract)
7. [Proposal Types](#proposal-types)
8. [ProposalFactory Contract](#proposalfactory-contract)
9. [Token Specifications](#token-specifications)
10. [Deployment Guide](#deployment-guide)
11. [Development Reference](#development-reference)

## Contract Architecture

MarketDAO consists of four main components:

1. **MarketDAO Contract**: The core contract that handles token management, treasury, and governance
2. **Proposal Base Contract**: Abstract contract that defines the proposal lifecycle
3. **Proposal Type Contracts**: Derived contracts that implement specific proposal functionality
4. **ProposalFactory Contract**: Factory that creates and manages proposals

### Inheritance Structure
```
ERC1155, ReentrancyGuard
      ↑
   MarketDAO
      ↑
   Proposal (Abstract)
      ↑
      ├── ResolutionProposal
      ├── TreasuryProposal
      ├── MintProposal
      └── TokenPriceProposal
```

## Security Features

MarketDAO implements multiple critical security features to protect against common DAO attacks and optimize gas costs:

### Governance Token Vesting

**Purpose**: Prevents a malicious actor from purchasing more governance tokens than the total outstanding supply and immediately voting themselves the entire treasury.

**Implementation**: (MarketDAO.sol:24-32, 96-106, 119-125)

When governance tokens are purchased through the `purchaseTokens()` function, they are subject to a configurable vesting period. The tokens are held by the buyer but cannot be used for governance until they vest.

- **Vesting Period**: Configured at DAO deployment via the `vestingPeriod` parameter (measured in blocks)
- **Vesting Schedules**: Each token purchase creates a new vesting schedule with an unlock block number
- **Automatic Cleanup**: Expired vesting schedules are automatically removed when transferring governance tokens
- **Schedule Consolidation**: Multiple purchases with the same unlock time are automatically merged
- **Schedule Limit**: Maximum 10 active vesting schedules per address (prevents DoS attacks)
- **Manual Cleanup**: Users can call `cleanupMyVestingSchedules()` to remove expired schedules anytime
- **Accurate Accounting**: Automatic cleanup maintains accurate `totalUnvestedGovernanceTokens` counter for quorum calculations
- **Vested Balance**: The `vestedBalance(address holder)` function calculates how many tokens are currently unlocked and available for governance
- **Governance Restrictions**: All governance functions (adding proposal support, voting) check `vestedBalance()` rather than total token balance
- **Frontend Display**: Dashboard shows total, vested, and unvested balances separately

**Attack Prevention**: An attacker who purchases a large number of tokens must wait for the vesting period to elapse before they can use those tokens to support proposals or vote, giving the DAO community time to respond.

**Note**: Initial token holders (from constructor) are not subject to vesting restrictions.

```solidity
struct VestingSchedule {
    uint256 amount;
    uint256 unlockBlock;
}

mapping(address => VestingSchedule[]) private vestingSchedules;

function vestedBalance(address holder) public view returns (uint256) {
    uint256 locked = 0;
    VestingSchedule[] storage schedules = vestingSchedules[holder];
    for (uint256 i = 0; i < schedules.length; i++) {
        if (block.number < schedules[i].unlockBlock) {
            locked += schedules[i].amount;
        }
    }
    return balanceOf(holder, GOVERNANCE_TOKEN_ID) - locked;
}
```

### Lazy Vote Token Distribution

**Purpose**: Reduces gas costs by avoiding the need to distribute voting tokens to all governance token holders when an election is triggered. Only users who actually participate in voting pay the gas to claim their voting tokens.

**Implementation**: (Proposal.sol:24, 122-137)

When an election is triggered, voting tokens are not minted upfront. Instead:

- **On-Demand Minting**: Users call `claimVotingTokens()` during the election period to receive their voting tokens
- **One-Time Claim**: The `hasClaimed` mapping ensures each holder can only claim once per election
- **Proportional Distribution**: Users receive voting tokens equal to their vested governance token balance at the time of claim
- **Vote Calculation**: The system calculates total possible votes based on all governance holders' vested balances, not just claimed tokens

**Gas Savings**: In a DAO with many token holders, only active participants pay gas to claim voting tokens, rather than the proposal creator paying gas to distribute to everyone upfront.

```solidity
mapping(address => bool) public hasClaimed;

function claimVotingTokens() external onlyDuringElection {
    require(!hasClaimed[msg.sender], "Already claimed voting tokens");

    uint256 vestedBal = dao.vestedBalance(msg.sender);
    require(vestedBal > 0, "No vested governance tokens to claim");

    hasClaimed[msg.sender] = true;
    dao.mintVotingTokens(msg.sender, votingTokenId, vestedBal);
}

function getClaimableAmount(address holder) external view returns (uint256) {
    if (!electionTriggered) return 0;
    if (hasClaimed[holder]) return 0;
    if (block.number >= electionStart + dao.electionDuration()) return 0;
    return dao.vestedBalance(holder);
}
```

### Purchase Restrictions (Optional)

**Purpose**: Prevents governance attacks where outsiders buy enough tokens to control the DAO by requiring community approval for all new members.

**Implementation**:

The purchase restriction feature can be enabled at deployment time (bit 1 of the flags parameter). When enabled:

- **Open mode (default)**: Anyone can purchase governance tokens directly
- **Restricted mode**: Only existing token holders can purchase additional tokens
- **Voting in new members**: When restricted, new members must be approved via mint proposals
- **Attack prevention**: Prevents hostile takeovers by requiring community approval for all new members
- **Configuration option**: Set `RESTRICT_PURCHASES = true` in the deployment script

**Use Cases:**
- **Investment clubs**: Members vote on admitting new partners
- **Private DAOs**: Closed membership with proposal-based expansion
- **Security-focused**: Extra protection against governance attacks

**Note**: The restriction check is based on current token balance (> 0), not historical holder status. If a holder transfers all tokens, they lose purchase privileges until they receive tokens again.

**Important**: The `restrictPurchasesToHolders` setting is configured at deployment and cannot be changed afterward. This ensures trust and predictability for DAO members.

### Join Request System

**Purpose**: Enables controlled membership growth while maintaining accessibility by allowing non-token holders to request membership.

**Implementation**:

Non-holders can submit join requests that create mint proposals for exactly 1 governance token:

- **Open to all**: Anyone without governance tokens can submit a join request
- **Proposal-based**: Join requests are mint proposals for exactly 1 governance token to the requester's address
- **Community voting**: Existing token holders vote on whether to admit new members
- **Self-introduction**: Requesters provide a description explaining who they are and why they want to join
- **One request per address**: Non-holders can only submit one join request (tracked via frontend localStorage)
- **Standard proposal flow**: Join requests follow the normal proposal lifecycle (support → election → execution)
- **Automatic restrictions**: Non-holders cannot create any other proposal types (resolution, treasury, token price)

**How It Works:**
1. Non-holder connects wallet and is presented with a "Request to Join DAO" interface
2. They submit a description introducing themselves
3. The system creates a mint proposal for 1 token to their address
4. Existing members see the request in the Proposals tab and can add support
5. Once support threshold is met, an election begins
6. Members vote YES or NO on admitting the new member
7. If approved, the new member receives 1 governance token and full DAO access
8. If rejected, they remain a non-holder but cannot submit another request

**Benefits:**
- **Controlled growth**: Community decides who joins
- **Prevents spam**: One request per address limits abuse
- **Transparency**: All join decisions are on-chain and voted on
- **Flexibility**: Works with both open and restricted purchase modes

**Note**: Token holders can still create mint proposals for any amount to any address. The 1-token restriction only applies to non-holders creating proposals for themselves.

### Snapshot-Based Voting Power

**Purpose**: Enable unlimited scalability without gas limit concerns by using O(1) snapshot creation.

**Implementation**:

MarketDAO uses a snapshot-based system for determining voting power:

- **O(1) snapshot creation**: Uses total vested supply instead of looping through all holders
- **Truly unlimited holders**: Tested with 10,000+ holders with constant gas costs
- **Accurate quorum**: Quorum calculated from vested supply only (unvested tokens cannot vote)
- **Fair voting**: Voting power frozen at election start, preventing mid-election manipulation
- **No gas limit concerns**: Election triggering cannot fail due to too many holders

**Gas Costs:**
- Election triggering: Constant ~280K gas regardless of holder count
- Proposal execution: O(1) regardless of holder count

## Security & Scalability

MarketDAO has been audited and hardened against common vulnerabilities:

### Security Features Summary
- ✅ **Reentrancy protection**: Transfer functions (`safeTransferFrom`, `safeBatchTransferFrom`) use ReentrancyGuard to prevent reentrancy during vote transfers and early termination
- ✅ **Factory-only proposal registration**: Only the official ProposalFactory can register proposals
- ✅ **Token holder restrictions**: Only addresses with vested governance tokens can create proposals (except join requests)
- ✅ **Join request validation**: Non-holders can only create mint proposals for exactly 1 token to their own address
- ✅ **Safe token transfers**: Uses OpenZeppelin's SafeERC20 and safeTransferFrom for all token operations
- ✅ **Basis points precision**: Thresholds use basis points (10000 = 100%) for 0.01% precision
- ✅ **Bounded gas costs**: All operations have predictable, capped gas costs

### Scalability Guarantees
- ✅ **Unlimited governance token holders**: O(1) snapshot using total supply enables 10,000+ participants
- ✅ **O(1) election triggering**: Constant 280K gas cost regardless of holder count
- ✅ **Automatic vesting cleanup**: Prevents unbounded array growth in vesting schedules
- ✅ **O(1) proposal execution**: Constant-time execution regardless of holder count
- ✅ **No gas limit concerns**: Election triggering cannot fail due to blockchain gas limits

### DoS Protection
- ✅ **No holder count limits**: O(1) snapshot prevents DoS from too many token holders
- ✅ **Vesting schedule limits**: Max 10 active schedules per address with auto-cleanup
- ✅ **Consolidation**: Automatic merging of schedules with same unlock time
- ✅ **Gas-bounded operations**: Election triggering uses constant gas regardless of holder count
- ✅ **Join request spam prevention**: Non-holders limited to one join request per address (frontend enforced)

## Known Limitations & Design Decisions

These are intentional design choices that should be understood before deployment:

### Purchase Restrictions Are Permanent

**Behavior**: The `restrictPurchasesToHolders` setting is configured at deployment and cannot be changed afterward. It's a fundamental characteristic of the DAO, not a governance parameter.

**Rationale**: This ensures trust and predictability. Members joining a restricted DAO know that membership requirements cannot be changed without redeploying. Similarly, open DAOs remain open permanently.

**Consideration**: Choose carefully based on your DAO's purpose:
- **Open**: Best for community DAOs, protocol governance, broad participation
- **Restricted**: Best for investment clubs, private organizations, security-focused DAOs

### Treasury Proposal Competition

**Behavior**: Multiple treasury proposals can be created requesting the same funds. Funds are only locked when a proposal reaches the support threshold and triggers an election. If proposal A locks the funds first, proposal B will fail when trying to start its election.

**Rationale**: Locking funds at proposal creation would enable trivial DoS attacks (spam proposals locking all treasury). Current design ensures only proposals with real community support (20%+ backing) can lock funds.

**Mitigation**: Community should coordinate on competing proposals. Frontend should display when multiple proposals request overlapping funds.

### Support Tracking After Token Transfers

**Behavior**: Support amounts are recorded when added but not automatically adjusted if users transfer their governance tokens afterward. Support only triggers elections - it does not affect voting outcomes.

**Why Not Critical**: Even if support is artificially inflated, winning an election still requires:
- 51% quorum participation from real token holders
- Majority YES votes based on actual token holdings at election start
- Attack cost (gas + token ownership) exceeds any benefit

**Mitigation**: Monitor for unusual support patterns. Set appropriate support thresholds to make attacks expensive.

### Fund Locking Gas Costs

**Behavior**: Functions that calculate available treasury balances (`getAvailableETH`, `getAvailableERC20`, etc.) iterate through all proposals with locked funds. Gas costs scale linearly with the number of concurrent treasury proposals in their election phase.

**Impact**: With many concurrent treasury proposals (50+), creating new treasury proposals or triggering elections could become expensive or potentially hit gas limits.

**Likelihood**: Low - Most DAOs will have fewer than 10 concurrent treasury elections at any time since elections are typically short (50 blocks).

**Mitigation**: If your DAO expects high concurrent treasury activity, consider shorter election durations or implementing a proposal limit.

## MarketDAO Contract

`MarketDAO.sol` implements the core DAO functionality, inheriting from OpenZeppelin's `ERC1155` and `ReentrancyGuard`.

### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `name` | `string` | Name of the DAO |
| `supportThreshold` | `uint256` | Percentage needed to trigger election |
| `quorumPercentage` | `uint256` | Percentage needed for valid election |
| `maxProposalAge` | `uint256` | Max age of proposal without election |
| `electionDuration` | `uint256` | Length of election in blocks |
| `allowMinting` | `bool` | Whether new governance tokens can be minted |
| `tokenPrice` | `uint256` | Price per token in wei (0 = direct sales disabled) |
| `vestingPeriod` | `uint256` | Vesting period for purchased tokens in blocks |
| `vestingSchedules` | `mapping(address => VestingSchedule[])` | Tracks vesting schedules per holder |
| `GOVERNANCE_TOKEN_ID` | `uint256` | Constant set to 0 |
| `nextVotingTokenId` | `uint256` | Next ID to use for voting tokens |
| `activeProposals` | `mapping(address => bool)` | Tracks active proposals |
| `isVoteAddress` | `mapping(address => bool)` | Marks addresses for voting |
| `voteAddressToProposal` | `mapping(address => address)` | Maps vote addresses to proposals |
| `hasTreasury` | `bool` | Whether the DAO has a treasury |
| `acceptsETH` | `bool` | Whether treasury accepts ETH |
| `acceptsERC20` | `bool` | Whether treasury accepts ERC20 tokens |
| `acceptsERC721` | `bool` | Whether treasury accepts ERC721 tokens |
| `acceptsERC1155` | `bool` | Whether treasury accepts ERC1155 tokens |
| `governanceTokenHolders` | `address[]` | List of governance token holders |
| `isGovernanceTokenHolder` | `mapping(address => bool)` | Tracks governance token holders |
| `tokenSupply` | `mapping(uint256 => uint256)` | Tracks token supply by ID |

### Constructor

```solidity
constructor(
    string memory _name,
    uint256 _supportThreshold,
    uint256 _quorumPercentage,
    uint256 _maxProposalAge,
    uint256 _electionDuration,
    bool _allowMinting,
    uint256 _tokenPrice,
    uint256 _vestingPeriod,
    string[] memory _treasuryConfig,
    address[] memory _initialHolders,
    uint256[] memory _initialAmounts
) ERC1155("")
```

### Key Functions

#### Token Management

```solidity
// Direct token purchase function (creates vesting schedule)
function purchaseTokens() external payable nonReentrant

// Calculate unlocked (vested) governance tokens for a holder
function vestedBalance(address holder) public view returns (uint256)

// Mint governance tokens (called by proposals)
function mintGovernanceTokens(address to, uint256 amount) external

// Mint voting tokens (called by proposals)
function mintVotingTokens(address to, uint256 tokenId, uint256 amount) external

// Get the next available voting token ID
function getNextVotingTokenId() external returns (uint256)

// Get total supply of a specific token ID
function totalSupply(uint256 tokenId) external view returns (uint256)

// Set the token price (called by proposals)
function setTokenPrice(uint256 newPrice) external
```

#### Governance Functions

```solidity
// Get list of governance token holders
function getGovernanceTokenHolders() external view returns (address[])

// Mark a proposal as active
function setActiveProposal(address proposal) external

// Clear an active proposal
function clearActiveProposal() external

// Register vote addresses
function registerVoteAddress(address voteAddr) external

// Check if a proposal is active
function isProposalActive(address proposal) external view returns (bool)
```

#### Treasury Functions

```solidity
// Receive ETH
receive() external payable

// Transfer ETH from treasury
function transferETH(address payable recipient, uint256 amount) external nonReentrant
```

#### Token Transfer Overrides

```solidity
// Override ERC1155 transfer functions to handle voting tokens
function safeTransferFrom(
    address from,
    address to,
    uint256 id,
    uint256 amount,
    bytes memory data
) public virtual override

function safeBatchTransferFrom(
    address from,
    address to,
    uint256[] memory ids,
    uint256[] memory amounts,
    bytes memory data
) public virtual override
```

## Proposal Base Contract

`Proposal.sol` implements the base proposal functionality, defining the lifecycle for all proposal types.

### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `dao` | `MarketDAO` | Reference to the parent DAO |
| `proposer` | `address` | Address that created the proposal |
| `createdAt` | `uint256` | Block number when proposal was created |
| `description` | `string` | Text description of the proposal |
| `supportTotal` | `uint256` | Total support accumulated |
| `support` | `mapping(address => uint256)` | Support by address |
| `electionTriggered` | `bool` | Whether election has been triggered |
| `electionStart` | `uint256` | Block number when election started |
| `votingTokenId` | `uint256` | Token ID for voting tokens |
| `yesVoteAddress` | `address` | Address for YES votes |
| `noVoteAddress` | `address` | Address for NO votes |
| `executed` | `bool` | Whether proposal has been executed |
| `hasClaimed` | `mapping(address => bool)` | Tracks which holders have claimed voting tokens |

### Constructor

```solidity
constructor(
    MarketDAO _dao,
    string memory _description
)
```

### Key Functions

```solidity
// Add support to a proposal (checks vested balance)
function addSupport(uint256 amount) external

// Remove support from a proposal
function removeSupport(uint256 amount) external

// Check if a proposal can trigger an election
function canTriggerElection() public view returns (bool)

// Internal function to trigger an election
function _triggerElection() internal

// Claim voting tokens during an election (lazy minting)
function claimVotingTokens() external

// View how many voting tokens an address can claim
function getClaimableAmount(address holder) external view returns (uint256)

// Check for early termination of election
function checkEarlyTermination() external virtual

// Execute a successful proposal
function execute() external

// Internal execution function (to be overridden)
function _execute() internal virtual

// Helper function to check if address is a vote address
function isVoteAddress(address addr) external view returns (bool)

// Check if election is currently active
function isElectionActive() external virtual view returns (bool)
```

## Proposal Types

### ResolutionProposal

Simple text-only proposal.

```solidity
constructor(
    MarketDAO _dao,
    string memory _description
) Proposal(_dao, _description)

function _execute() internal override
```

### TreasuryProposal

Transfers assets from the DAO treasury.

```solidity
// State variables
address public recipient;
uint256 public amount;
address public token;
uint256 public tokenId;

constructor(
    MarketDAO _dao,
    string memory _description,
    address _recipient,
    uint256 _amount,
    address _token,
    uint256 _tokenId
) Proposal(_dao, _description)

function _execute() internal override
```

### MintProposal

Creates new governance tokens.

```solidity
// State variables
address public recipient;
uint256 public amount;

constructor(
    MarketDAO _dao,
    string memory _description,
    address _recipient,
    uint256 _amount
) Proposal(_dao, _description)

function _execute() internal override
```

### TokenPriceProposal

Changes the price for direct governance token purchases.

```solidity
// State variables
uint256 public newPrice;

constructor(
    MarketDAO _dao,
    string memory _description,
    uint256 _newPrice
) Proposal(_dao, _description)

function _execute() internal override
```

## ProposalFactory Contract

`ProposalFactory.sol` facilitates the creation of different proposal types.

### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `dao` | `MarketDAO` | Reference to the parent DAO |
| `proposals` | `mapping(uint256 => address)` | Maps indices to proposal addresses |
| `proposalCount` | `uint256` | Number of proposals created |

### Constructor

```solidity
constructor(MarketDAO _dao)
```

### Key Functions

```solidity
// Create a text-only resolution proposal
function createResolutionProposal(
    string memory description
) external returns (ResolutionProposal)

// Create a treasury transfer proposal
function createTreasuryProposal(
    string memory description,
    address recipient,
    uint256 amount,
    address token,
    uint256 tokenId
) external returns (TreasuryProposal)

// Create a token minting proposal
function createMintProposal(
    string memory description,
    address recipient,
    uint256 amount
) external returns (MintProposal)

// Create a token price update proposal
function createTokenPriceProposal(
    string memory description,
    uint256 newPrice
) external returns (TokenPriceProposal)

// Get a proposal by index
function getProposal(uint256 index) external view returns (address)
```

## Token Specifications

### Governance Tokens (ID 0)

- **Usage**: Create and support proposals, receive voting tokens
- **Distribution**: Initial allocation, direct purchase, or mint proposals
- **Transfer**: Freely transferable ERC1155 tokens
- **Vesting**: Purchased tokens are subject to vesting period before they can be used for governance
- **Governance Rights**: Only vested (unlocked) tokens can be used to support proposals or participate in voting

### Voting Tokens (IDs 1+)

- **Creation**: One unique token ID per election
- **Distribution**: Lazy minting - holders must claim tokens during election period by calling `claimVotingTokens()`
- **Amount**: 1:1 with the holder's vested governance token balance at time of claim
- **Usage**: Sent to YES/NO addresses to vote
- **Transfer**: Freely transferable during election period
- **Lifecycle**: Become worthless after election concludes
- **Gas Efficiency**: Only participants who vote pay gas to claim their tokens

## Deployment Guide

### Prerequisites

- Solidity ^0.8.20
- OpenZeppelin Contracts
- Foundry or similar deployment tool

### Deployment Steps

1. Deploy the MarketDAO contract with desired parameters
2. Deploy the ProposalFactory contract, passing the MarketDAO address
3. Verify contracts on block explorer (optional)

### Constructor Parameters

When deploying the MarketDAO contract, you'll need to provide:

```solidity
string _name,                  // Name of the DAO
uint256 _supportThreshold,     // Percentage needed to trigger election in basis points (e.g., 2000 = 20%)
uint256 _quorumPercentage,     // Percentage needed for valid election in basis points (e.g., 5100 = 51%)
uint256 _maxProposalAge,       // Max age of proposal in blocks
uint256 _electionDuration,     // Length of election in blocks
uint256 _flags,                // Bitfield for boolean options (bit 0: allow minting, bit 1: restrict purchases)
uint256 _tokenPrice,           // Price per token in wei (0 = direct sales disabled)
uint256 _vestingPeriod,        // Vesting period for purchased tokens in blocks (0 = no vesting)
string[] _treasuryConfig,      // Array of supported asset types (e.g., ["ETH", "ERC20"])
address[] _initialHolders,     // Array of initial token holder addresses
uint256[] _initialAmounts      // Array of initial token amounts
```

**Note on Basis Points**: All percentage parameters use basis points for precision:
- 10000 = 100%
- 5100 = 51%
- 2000 = 20%
- 250 = 2.5%

This allows for fractional percentages with 0.01% precision.

**Configuring Flags**: The deployment script (`script/Deploy.s.sol`) provides boolean configuration options that are automatically converted to the flags bitfield:
```solidity
bool constant ALLOW_MINTING = true;            // Enable governance token minting
bool constant RESTRICT_PURCHASES = false;      // Allow anyone to purchase tokens
```

The `buildFlags()` function handles the conversion automatically.

## Development Reference

### Environment Setup

```bash
# Clone the repository
git clone https://github.com/evronm/marketDAO
cd marketDAO

# Install dependencies
forge install

# Build contracts
forge build

# Run tests
forge test

# Format code
forge fmt

# Deploy locally
./deploy.sh
```

### GitHub Repository

- **Source Code**: [https://github.com/evronm/marketDAO](https://github.com/evronm/marketDAO)
- **Contracts**: Located in the `src/` directory

### Key Dependencies

- **OpenZeppelin Contracts**:
  - ERC1155
  - ReentrancyGuard
  - SafeERC20
  - IERC20
  - IERC721
  - IERC1155
