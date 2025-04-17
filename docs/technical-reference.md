---
layout: page
title: MarketDAO Technical Reference
permalink: /docs/technical-reference/
---
# MarketDAO Technical Reference

## Table of Contents

1. [Contract Architecture](#contract-architecture)
2. [MarketDAO Contract](#marketdao-contract)
3. [Proposal Base Contract](#proposal-base-contract)
4. [Proposal Types](#proposal-types)
5. [ProposalFactory Contract](#proposalfactory-contract)
6. [Token Specifications](#token-specifications)
7. [Deployment Guide](#deployment-guide)
8. [Development Reference](#development-reference)

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
    string[] memory _treasuryConfig,
    address[] memory _initialHolders,
    uint256[] memory _initialAmounts
) ERC1155("")
```

### Key Functions

#### Token Management

```solidity
// Direct token purchase function
function purchaseTokens() external payable nonReentrant

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

### Constructor

```solidity
constructor(
    MarketDAO _dao,
    string memory _description
)
```

### Key Functions

```solidity
// Add support to a proposal
function addSupport(uint256 amount) external

// Remove support from a proposal
function removeSupport(uint256 amount) external

// Check if a proposal can trigger an election
function canTriggerElection() public view returns (bool)

// Internal function to trigger an election
function _triggerElection() internal

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

### Voting Tokens (IDs 1+)

- **Creation**: Minted at election start, one unique ID per election
- **Distribution**: 1:1 with governance tokens at election trigger
- **Usage**: Sent to YES/NO addresses to vote
- **Transfer**: Freely transferable during election period
- **Lifecycle**: Become worthless after election concludes

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
uint256 _supportThreshold,     // Percentage needed to trigger election (e.g., 15)
uint256 _quorumPercentage,     // Percentage needed for valid election (e.g., 25)
uint256 _maxProposalAge,       // Max age of proposal in blocks
uint256 _electionDuration,     // Length of election in blocks
bool _allowMinting,            // Whether new governance tokens can be minted
uint256 _tokenPrice,           // Price per token in wei (0 = direct sales disabled)
string[] _treasuryConfig,      // Array of supported asset types (e.g., ["ETH", "ERC20"])
address[] _initialHolders,     // Array of initial token holder addresses
uint256[] _initialAmounts      // Array of initial token amounts
```

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

### Current Deployment

- **Frontend**: [https://evronm.github.io/marketDAO/index.html](https://evronm.github.io/marketDAO/index.html)
- **DAO Contract**: 0xf188d689d78b58b9d3e1a841a9b9afb8f92ddf55 (Polygon Amoy testnet)
- **Factory Contract**: 0xc609fa60239116ecee53af12f733eb0214d7b1ad (Polygon Amoy testnet)

### Key Dependencies

- **OpenZeppelin Contracts**:
  - ERC1155
  - ReentrancyGuard
  - SafeERC20
  - IERC20
  - IERC721
  - IERC1155
