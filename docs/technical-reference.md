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
8. [DistributionRedemption Contract](#distributionredemption-contract)
9. [ProposalFactory Contract](#proposalfactory-contract)
10. [Token Specifications](#token-specifications)
11. [Deployment Guide](#deployment-guide)
12. [Development Reference](#development-reference)

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

Key protections:
- Unvested tokens cannot be transferred
- Unvested tokens cannot be used for proposal support
- Unvested tokens cannot be used for voting (voting token claims only include vested balance)
- Initial token distribution (at DAO creation) bypasses vesting

**Vesting Schedule Management:**
- Maximum 10 vesting schedules per address (DoS protection)
- Automatic consolidation of schedules with the same unlock time
- Automatic cleanup of expired schedules on claim
- Efficient O(1) tracking of total unvested tokens

### Lazy Vote Token Distribution

**Purpose**: Minimizes gas costs when elections are triggered by not automatically minting voting tokens for all holders.

**Implementation**: (Proposal.sol:182-194)

When an election starts, voting tokens are NOT automatically distributed. Instead:
- Each holder must claim their voting tokens before participating
- Voting token amount equals the holder's vested governance tokens at election start
- One-time claim per address per election

Benefits:
- Proposer doesn't pay gas to mint tokens for all holders
- Only active participants incur gas costs
- Scales to unlimited number of governance token holders

### Purchase Restrictions (Optional)

**Purpose**: Prevents hostile takeovers by limiting token purchases to existing holders.

**Implementation**: (MarketDAO.sol:38, 215)

When enabled (`RESTRICT_PURCHASES = true`), only addresses with an existing governance token balance can purchase additional tokens. Non-holders must submit join requests.

### Join Request System

**Purpose**: Provides a democratic path for new members to join restricted DAOs.

**Implementation**: (ProposalFactory.sol:69-76)

Non-holders can create a special mint proposal requesting exactly 1 governance token to their own address. This proposal goes through the normal support and voting process, allowing existing members to vet new applicants.

### Snapshot-Based Voting Power

**Purpose**: Ensures unlimited scalability by avoiding iteration over all token holders.

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

MarketDAO has been audited by **[Hashlock Pty Ltd](https://hashlock.com/audits/marketdao)** (January 2026). All HIGH severity issues have been resolved.

<p style="text-align: center; margin: 20px 0;">
    <a href="https://hashlock.com/audits/marketdao" target="_blank" rel="noopener">
        <img src="{{ site.baseurl }}/assets/images/hashlock-badge.png" alt="Audited by Hashlock" style="max-width: 200px;">
    </a>
</p>
<p style="text-align: center;">
    <a href="https://hashlock.com/audits/marketdao" target="_blank" rel="noopener" class="cta-button">View Full Audit Report</a>
</p>

### Security Features Summary

- ✅ **Reentrancy protection**: Transfer functions (`safeTransferFrom`, `safeBatchTransferFrom`) use ReentrancyGuard to prevent reentrancy during vote transfers and early termination
- ✅ **Governance token locking**: Tokens are locked when used for proposal support or voting claims, preventing double-counting (H-03/H-04 fix)
- ✅ **Distribution token locking**: Tokens are locked when registering for distributions, preventing double-claim attacks (H-02 fix)
- ✅ **Operator voting restrictions**: Election-ended checks apply to all transfers, not just direct transfers (H-05 fix)
- ✅ **Pro-rata distributions**: Distribution claims use proportional calculations to prevent pool exhaustion (M-01 fix)
- ✅ **Controlled pool funding**: Only the proposal contract can mark distribution pools as funded, preventing griefing attacks (M-01 fix)
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

### Audit-Identified Limitations

- **M-02 (Stale Vested Supply)**: `getTotalVestedSupply()` may be slightly understated if users don't claim vested tokens. This makes governance slightly easier (not harder) and self-corrects through normal usage.

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

**Rationale**: Tracking support changes would require expensive on-transfer hooks. Since support only determines whether an election starts (not who wins), the security impact is minimal.

**Mitigation**: Proposals expire after `maxProposalAge` blocks, limiting the window for this edge case.

### Fund Locking Gas Costs

**Behavior**: The `getAvailableBalance()` function iterates through all proposals with locked funds. Gas costs scale linearly with concurrent treasury proposals.

**Rationale**: Treasury proposals are relatively rare (compared to token transfers), and the iteration only includes proposals that have triggered elections and locked funds.

**Mitigation**: In practice, DAOs rarely have more than 2-3 concurrent treasury proposals in election. The gas cost remains manageable.

## MarketDAO Contract

`MarketDAO.sol` is the core contract that inherits from ERC1155 and ReentrancyGuard.

### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `GOVERNANCE_TOKEN_ID` | `uint256 constant` | Always 0, identifies governance tokens |
| `name` | `string` | Name of the DAO |
| `supportThreshold` | `uint256` | Percentage needed to trigger election (basis points) |
| `quorumPercentage` | `uint256` | Percentage needed for valid election (basis points) |
| `maxProposalAge` | `uint256` | Max age of proposal in blocks |
| `electionDuration` | `uint256` | Length of election in blocks |
| `flags` | `uint256` | Bitfield for boolean options |
| `tokenPrice` | `uint256` | Price per token in wei |
| `vestingPeriod` | `uint256` | Vesting period for purchased tokens in blocks |
| `factory` | `address` | Address of the ProposalFactory |
| `activeProposals` | `mapping(address => bool)` | Tracks active proposals |
| `distributionLock` | `mapping(address => uint256)` | Tracks locked tokens for distribution registration (H-02 fix) |
| `governanceLock` | `mapping(address => uint256)` | Tracks locked tokens for proposal support/voting (H-03/H-04 fix) |
| `activeRedemptionContract` | `address` | Currently active distribution redemption contract |

### Key Functions

```solidity
// Purchase governance tokens (subject to vesting)
function purchaseTokens(uint256 amount) external payable

// Claim vested tokens
function claimVestedTokens() external

// Get vested balance for an address
function vestedBalance(address holder) public view returns (uint256)

// Get transferable balance (vested minus all locks)
function transferableBalance(address holder) public view returns (uint256)

// Get total vested supply (for quorum calculation)
function getTotalVestedSupply() public view returns (uint256)

// Lock/unlock governance tokens for distribution (H-02 fix)
function lockForDistribution(address user, uint256 amount) external
function unlockForDistribution(address user) external

// Add/remove governance locks for proposals (H-03/H-04 fix)
function addGovernanceLock(address user, uint256 amount) external
function removeGovernanceLock(address user, uint256 amount) external
```

## Proposal Base Contract

`Proposal.sol` is an abstract contract that defines the proposal lifecycle.

### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `dao` | `MarketDAO` | Reference to the parent DAO |
| `description` | `string` | Proposal description |
| `creationBlock` | `uint256` | Block when proposal was created |
| `support` | `mapping(address => uint256)` | Support amounts by address |
| `supportTotal` | `uint256` | Total support amount |
| `supportLocked` | `mapping(address => uint256)` | Locked tokens for support (H-03 fix) |
| `votingLocked` | `mapping(address => uint256)` | Locked tokens for voting (H-04 fix) |
| `electionTriggered` | `bool` | Whether election has started |
| `electionStart` | `uint256` | Block when election started |
| `snapshotTotalVotes` | `uint256` | Total voting power at election start |
| `executed` | `bool` | Whether proposal has been executed |
| `votingTokenId` | `uint256` | ID of voting tokens for this election |
| `yesVoteAddress` | `address` | Address representing YES votes |
| `noVoteAddress` | `address` | Address representing NO votes |
| `hasClaimed` | `mapping(address => bool)` | Tracks who has claimed voting tokens |

### Key Functions

```solidity
// Add support to trigger election (locks tokens - H-03 fix)
function addSupport(uint256 amount) external

// Remove support before election starts (unlocks tokens)
function removeSupport(uint256 amount) external

// Claim voting tokens during election (locks tokens - H-04 fix)
function claimVotingTokens() external

// Execute passed proposal after election ends
function execute() external

// Mark proposal as failed (quorum not met or rejected)
function fail() external

// Check if proposal has expired
function isExpired() public view returns (bool)

// Check if election is currently active
function isElectionActive() public view returns (bool)

// Check if proposal is resolved (executed, failed, or expired)
function isResolved() public view returns (bool)

// Release all governance locks held by this proposal for the caller (H-03/H-04 fix)
function releaseProposalLocks() external

// Get total locked amount for a user by this proposal
function getLockedAmount(address user) external view returns (uint256)

// Check for early termination when majority reached
function checkEarlyTermination() public
```

## Proposal Types

### ResolutionProposal

Simple text-only proposals for governance decisions that don't require on-chain actions.

```solidity
constructor(
    MarketDAO _dao,
    string memory _description
) Proposal(_dao, _description)

function _execute() internal override
// No-op: resolution proposals are purely symbolic
```

### TreasuryProposal

Transfers assets from the DAO treasury to a recipient.

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

### DistributionProposal

Creates proportional distributions of treasury assets to all governance token holders.

```solidity
// State variables
address public token;           // Token address (address(0) for ETH)
uint256 public tokenId;         // Token ID for ERC1155 (0 for ETH/ERC20)
uint256 public amountPerToken;  // Target amount per governance token
DistributionRedemption public redemptionContract;

constructor(
    MarketDAO _dao,
    string memory _description,
    address _token,
    uint256 _tokenId,
    uint256 _amountPerToken
) Proposal(_dao, _description)

// Register for distribution before election ends
function registerForDistribution() external

function _execute() internal override
```

## DistributionRedemption Contract

`DistributionRedemption.sol` is a standalone contract created by DistributionProposal to manage proportional asset distributions.

### Security Features

- **H-02 FIX**: Locks governance tokens in MarketDAO when users register, preventing the same tokens from being used to register multiple addresses for the same distribution
- **M-01 FIX**: Uses pro-rata distribution to ensure all registered users can claim. Each user receives: `(userShares / totalRegisteredShares) * actualPoolBalance`. This means `amountPerGovernanceToken` is a TARGET, not a guarantee.
- **M-01 FIX (Part 2)**: Only the proposal can mark the pool as funded via `markPoolFunded()`. This prevents griefing attacks where attackers send dust to snapshot a tiny balance before real funds arrive.

### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `proposal` | `address` | The distribution proposal that created this contract |
| `dao` | `IMarketDAO` | Reference to the MarketDAO contract |
| `token` | `address` | Token address (address(0) for ETH) |
| `tokenId` | `uint256` | Token ID for ERC1155 (0 for ETH/ERC20) |
| `amountPerGovernanceToken` | `uint256` | Target amount per governance token |
| `registeredBalance` | `mapping(address => uint256)` | Registered governance token balances |
| `totalRegisteredGovernanceTokens` | `uint256` | Total registered governance tokens |
| `hasClaimed` | `mapping(address => bool)` | Tracks which addresses have claimed |
| `poolFunded` | `bool` | Whether the pool has been officially funded |
| `totalPoolBalance` | `uint256` | Snapshot of pool balance when funded |

### Key Functions

```solidity
// Register a user for distribution (only callable by proposal)
function registerClaimant(address user, uint256 governanceTokenBalance) external

// Claim distributed funds (pro-rata calculation)
function claim() external

// Release lock without claiming (after distribution ends)
function releaseLock() external

// Mark pool as funded (only callable by proposal)
function markPoolFunded() external

// View claimable amount (pro-rata based on actual pool balance)
function getClaimableAmount(address holder) external view returns (uint256)

// Check if address has registered
function isRegistered(address holder) external view returns (bool)

// Check if distribution has ended
function isDistributionEnded() external view returns (bool)
```

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
constructor(
    MarketDAO _dao,
    address _resolutionImpl,
    address _treasuryImpl,
    address _mintImpl,
    address _parameterImpl,
    address _distributionImpl
)
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
    uint256 amount
) external returns (TreasuryProposal)

// Create a mint proposal
function createMintProposal(
    string memory description,
    address recipient,
    uint256 amount
) external returns (MintProposal)

// Create a parameter change proposal
function createParameterProposal(
    string memory description,
    ParameterProposal.ParameterType parameterType,
    uint256 newValue
) external returns (ParameterProposal)

// Create a distribution proposal
function createDistributionProposal(
    string memory description,
    address token,
    uint256 tokenId,
    uint256 amountPerToken
) external returns (DistributionProposal)

// Get proposal by index
function getProposal(uint256 index) external view returns (address)
```

## Token Specifications

### Governance Tokens (Token ID 0)

- Standard ERC1155 token with ID 0
- Represent voting power and proposal creation rights
- Subject to vesting when purchased
- Can be locked for distribution registration (H-02 fix)
- Can be locked for proposal support and voting (H-03/H-04 fix)
- Transferable only when vested AND not locked

### Voting Tokens (Token IDs 1+)

- Created for each election
- Claimed from vested governance token balance
- Fully transferable during election period
- Transfer to YES/NO addresses counts as voting
- Election-ended checks apply to ALL transfers, not just direct ones (H-05 fix)

## Deployment Guide

### Prerequisites

- Foundry installed (`curl -L https://foundry.paradigm.xyz | bash`)
- Private key with funds for deployment
- RPC URL for target network

### Deployment Steps

1. Clone the repository and install dependencies
2. Configure deployment parameters in `script/Deploy.s.sol`
3. Deploy the MarketDAO contract with desired parameters
4. Deploy the ProposalFactory contract, passing the MarketDAO address
5. Verify contracts on block explorer (optional)

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
