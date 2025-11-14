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
      ├── DistributionProposal
      ├── MintProposal
      └── ParameterProposal

   DistributionRedemption (Standalone Contract)
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

**Important**: The `restrictPurchasesToHolders` setting (bit 1 of the flags parameter) can be changed through Parameter Proposals, allowing DAOs to evolve their membership model democratically. However, this is a fundamental characteristic that should only be changed with broad community consensus.

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

### Distribution Proposals (Proportional Distributions)

**Purpose**: Enable fair, proportional distributions of assets to all token holders based on their governance token holdings.

**Implementation**:

Distribution Proposals allow DAOs to distribute assets (ETH, ERC20, ERC1155) proportionally to token holders:

- **Proportional distribution**: Each holder receives an amount based on their registered token balance
- **Registration system**: Token holders must register before the election to receive distributions
- **Redemption contract**: Approved distributions transfer funds to a separate DistributionRedemption contract where users claim individually
- **Claimable by holders**: Each registered holder claims their share when ready (no batch distribution gas costs)
- **Asset support**: Works with ETH, ERC20, and ERC1155 tokens
- **Flexible timing**: Users can claim at any time after proposal execution

**How It Works:**
1. **Create proposal**: Specify the asset type, amount per governance token, and description
2. **Registration phase**: Token holders register during the support/election phases to be included
3. **Voting**: Standard election process with support threshold and voting
4. **Execution**: On approval, funds transfer to a DistributionRedemption contract
5. **Claiming**: Registered holders claim their proportional share from the redemption contract

**Example Use Cases:**
- **Revenue sharing**: Distribute profits or royalties proportionally to all token holders
- **Dividend payments**: Regular distributions based on treasury performance
- **Airdrops**: Distribute ERC20 tokens or NFTs proportionally
- **Liquidation**: Fair distribution when winding down a DAO

**Gas Efficiency:**
- No upfront distribution to all holders (would be prohibitively expensive)
- Each holder claims individually, paying their own gas
- Scales to unlimited holders without gas limit concerns

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

### Immutable vs. Changeable Parameters

**Immutable (Set at Deployment)**:
- **DAO Name**: Cannot be changed after deployment
- **Treasury Configuration**: Which asset types (ETH, ERC20, ERC721, ERC1155) the DAO accepts

**Changeable via Parameter Proposals**:
- Support Threshold
- Quorum Percentage
- Max Proposal Age
- Election Duration
- Vesting Period
- Token Price
- Flags (Allow Minting, Restrict Purchases, Mint to Purchase)

**Rationale**: Immutable parameters ensure trust and predictability. Members joining a DAO know that fundamental characteristics (name, treasury capabilities) cannot be changed without redeploying. All governance-related parameters can be modified democratically as the DAO evolves.

**Important Note on Purchase Restrictions**: The "Restrict Purchases" flag can be enabled or disabled through Parameter Proposals (democratic voting). This allows DAOs to evolve their membership model:
- **Open mode**: Anyone can purchase governance tokens directly
- **Restricted mode**: Only existing token holders can purchase additional tokens

While this flag is changeable, it affects the fundamental nature of DAO membership and should only be modified with broad consensus.

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
| `flags` | `uint256` | Bitfield for boolean configuration (bit 0: allow minting, bit 1: restrict purchases, bit 2: mint to purchase) |
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
| `totalUnvestedGovernanceTokens` | `uint256` | Tracks total unvested tokens for accurate quorum |
| `lockedFunds` | `mapping` | Tracks funds locked by treasury proposals |
| `flags` | `uint256` | Bitfield for boolean configuration options |

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

// Parameter setter functions (called by ParameterProposal)
function setSupportThreshold(uint256 newThreshold) external
function setQuorumPercentage(uint256 newQuorum) external
function setMaxProposalAge(uint256 newAge) external
function setElectionDuration(uint256 newDuration) external
function setVestingPeriod(uint256 newPeriod) external
function setTokenPrice(uint256 newPrice) external
function setFlags(uint256 newFlags) external

// Get total vested supply (O(1) snapshot for quorum calculations)
function getTotalVestedSupply() external view returns (uint256)
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

### DistributionProposal

Distributes assets proportionally to all registered token holders.

```solidity
// State variables
AssetType public assetType;        // ETH, ERC20, or ERC1155
address public tokenAddress;        // Token contract address (if applicable)
uint256 public tokenId;             // Token ID (for ERC1155)
uint256 public amountPerToken;      // Amount to distribute per governance token
address public redemptionContract;  // DistributionRedemption contract address
mapping(address => bool) public hasRegistered;  // Tracks registered holders

constructor(
    MarketDAO _dao,
    string memory _description,
    AssetType _assetType,
    address _tokenAddress,
    uint256 _tokenId,
    uint256 _amountPerToken
) Proposal(_dao, _description)

// Register to receive distribution
function registerForDistribution() external

// Check if address has registered
function isRegistered(address holder) external view returns (bool)

function _execute() internal override
```

**Key Features:**
- Token holders register during support/election phases
- On execution, creates a DistributionRedemption contract
- Transfers funds to redemption contract for claiming
- Each registered holder can claim their proportional share at any time

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

### ParameterProposal

Changes DAO configuration parameters through governance.

```solidity
// State variables
enum ParameterType {
    SupportThreshold,      // 0: Percentage needed to trigger election (basis points, > 0 and <= 10000)
    QuorumPercentage,      // 1: Percentage needed for valid election (basis points, >= 100 and <= 10000)
    MaxProposalAge,        // 2: Max age of proposal without election (blocks, > 0)
    ElectionDuration,      // 3: Length of election in blocks (> 0)
    VestingPeriod,         // 4: Vesting period for purchased tokens (blocks, >= 0)
    TokenPrice,            // 5: Price per token in wei (> 0, set to 0 to disable direct sales)
    Flags                  // 6: Bitfield for boolean options (<= 7, bits 0-2 only)
}

ParameterType public parameterType;
uint256 public newValue;

constructor(
    MarketDAO _dao,
    string memory _description,
    ParameterType _parameterType,
    uint256 _newValue
) Proposal(_dao, _description)

function _execute() internal override
```

**Parameter Types:**
- **SupportThreshold** (0): Changes the percentage of governance tokens needed to trigger an election (basis points, must be > 0 and <= 10000)
- **QuorumPercentage** (1): Changes the percentage participation needed for a valid election (basis points, must be >= 100 and <= 10000, minimum 1%)
- **MaxProposalAge** (2): Changes how long a proposal can exist before expiring (blocks, must be > 0)
- **ElectionDuration** (3): Changes the length of the voting period (blocks, must be > 0)
- **VestingPeriod** (4): Changes the vesting period for purchased governance tokens (blocks, can be 0 to disable vesting)
- **TokenPrice** (5): Changes the price for direct token purchases (wei, must be > 0, or 0 to disable direct sales)
- **Flags** (6): Changes boolean configuration options (must be <= 7, only bits 0-2 are valid)
  - Bit 0: Allow minting (can new governance tokens be minted)
  - Bit 1: Restrict purchases (limit to existing holders)
  - Bit 2: Mint to purchase (mint new tokens vs transfer from treasury)

Each parameter type has built-in validation to prevent invalid configurations.

## DistributionRedemption Contract

`DistributionRedemption.sol` is a standalone contract created by DistributionProposal to manage proportional asset distributions.

### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `assetType` | `AssetType` | Type of asset being distributed (ETH, ERC20, ERC1155) |
| `tokenAddress` | `address` | Contract address for token distributions |
| `tokenId` | `uint256` | Token ID for ERC1155 distributions |
| `amountPerToken` | `uint256` | Amount distributed per governance token |
| `registeredHolders` | `mapping(address => uint256)` | Maps registered addresses to their token balances |
| `hasClaimed` | `mapping(address => bool)` | Tracks which addresses have claimed |

### Constructor

```solidity
constructor(
    AssetType _assetType,
    address _tokenAddress,
    uint256 _tokenId,
    uint256 _amountPerToken,
    address[] memory _holders,
    uint256[] memory _balances
)
```

### Key Functions

```solidity
// Claim proportional distribution
function claim() external

// View claimable amount for an address
function getClaimableAmount(address holder) external view returns (uint256)

// Check if address has claimed
function hasClaimed(address holder) external view returns (bool)
```

**Security Features:**
- Each holder can only claim once
- Claims are proportional to registered governance token balance
- Supports ETH, ERC20, and ERC1155 assets
- Gas efficient: holders pay their own claim gas

## ProposalFactory Contract

`ProposalFactory.sol` facilitates the creation of different proposal types.

### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `dao` | `MarketDAO` | Reference to the parent DAO |
| `proposals` | `mapping(uint256 => address)` | Maps indices to proposal addresses |
| `proposalCount` | `uint256` | Number of proposals created |
| `resolutionImpl` | `address` | Implementation contract for resolution proposals |
| `treasuryImpl` | `address` | Implementation contract for treasury proposals |
| `distributionImpl` | `address` | Implementation contract for distribution proposals |
| `mintImpl` | `address` | Implementation contract for mint proposals |
| `parameterImpl` | `address` | Implementation contract for parameter proposals |

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

// Create a distribution proposal
function createDistributionProposal(
    string memory description,
    AssetType assetType,
    address tokenAddress,
    uint256 tokenId,
    uint256 amountPerToken
) external returns (DistributionProposal)

// Create a token minting proposal
function createMintProposal(
    string memory description,
    address recipient,
    uint256 amount
) external returns (MintProposal)

// Create a parameter change proposal
function createParameterProposal(
    string memory description,
    ParameterType parameterType,
    uint256 newValue
) external returns (ParameterProposal)

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
