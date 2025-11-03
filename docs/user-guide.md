---
layout: page
title: MarketDAO User Guide
permalink: /docs/user-guide/
---

## Table of Contents

1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
   - [Connecting Your Wallet](#connecting-your-wallet)
   - [Understanding the Dashboard](#understanding-the-dashboard)
3. [Join Request System (For Non-Members)](#join-request-system-for-non-members)
   - [How to Submit a Join Request](#how-to-submit-a-join-request)
   - [Join Request Process](#join-request-process)
4. [Governance Tokens](#governance-tokens)
   - [Acquiring Governance Tokens](#acquiring-governance-tokens)
   - [Checking Your Balance](#checking-your-balance)
5. [Creating Proposals](#creating-proposals)
   - [Resolution Proposals](#resolution-proposals)
   - [Treasury Proposals](#treasury-proposals)
   - [Mint Proposals](#mint-proposals)
   - [Parameter Proposals](#parameter-proposals)
6. [Supporting Proposals](#supporting-proposals)
7. [Participating in Elections](#participating-in-elections)
   - [Understanding the Election Process](#understanding-the-election-process)
   - [Voting on Proposals](#voting-on-proposals)
   - [Trading Voting Tokens](#trading-voting-tokens)
8. [Viewing Proposal History](#viewing-proposal-history)
9. [FAQ](#faq)

## Introduction

MarketDAO is a governance framework that brings market forces to group decisions. The key innovation is that **voting rights can be bought and sold during elections**, allowing market forces to influence governance outcomes. This user guide will walk you through all aspects of participating in MarketDAO governance.

## Getting Started

### Deploying a MarketDAO Instance

To use MarketDAO, you'll need to deploy your own instance:

1. Clone the repository from [https://github.com/evronm/marketDAO](https://github.com/evronm/marketDAO)
2. Follow the deployment instructions in the [Technical Reference]({{ site.baseurl }}/docs/technical-reference/#deployment-guide)
3. Use the included frontend to interact with your deployed contracts

> **Note:** The repository includes a frontend application that is compatible with the current contracts. This user guide describes the features and workflow using that frontend.

### Understanding the Dashboard

After connecting your wallet, you'll see the Dashboard, which displays:

- **DAO Information** - Name, your token balance, total token supply, token price, etc.
- **Token Purchase** - Interface to buy governance tokens
- **Navigation Tabs** - Access to Proposals, Elections, and History sections

## Join Request System (For Non-Members)

If you don't have governance tokens yet, you can request to join the DAO through the Join Request System. This feature allows potential members to introduce themselves and request admission to the DAO.

### How to Submit a Join Request

1. **Connect Your Wallet**: Connect to the DAO interface with a wallet that does not hold governance tokens
2. **Access Join Request Interface**: You'll see a "Request to Join DAO" interface instead of the standard proposal creation options
3. **Introduce Yourself**: Provide a description explaining:
   - Who you are
   - Why you want to join the DAO
   - What you can contribute to the community
4. **Submit Request**: Click the submit button to create your join request
5. **One Request Per Address**: You can only submit one join request per wallet address

### Join Request Process

Your join request follows the standard proposal lifecycle:

1. **Support Phase**: Existing members review your request and can add support
   - Your request needs to reach the support threshold (typically 20% of governance tokens)
   - Members evaluate your introduction and decide if they support your admission

2. **Election Phase**: Once support threshold is met, an election begins
   - Existing members vote YES or NO on admitting you
   - The election must meet quorum requirements (typically 51% participation)

3. **Outcome**:
   - **If Approved**: You receive 1 governance token and become a full member with all rights and privileges
   - **If Rejected**: Your request is declined, but you remain able to observe the DAO

**Important Notes:**
- Non-holders can only create join requests (1 token to yourself)
- Token holders can create regular mint proposals for any amount to any address
- Join requests work in both open and restricted purchase mode DAOs
- Even if rejected, the on-chain record shows you attempted to participate

### Purchase Restrictions

Some DAOs may be configured with purchase restrictions:

- **Open Mode (default)**: Anyone can purchase governance tokens directly without approval
- **Restricted Mode**: Only existing token holders can purchase additional tokens
  - New members must be approved via join requests or mint proposals
  - This protects against hostile takeovers by outsiders
  - Best for investment clubs, private organizations, and security-focused DAOs

Check the DAO's configuration to see which mode is active. If you're unsure, try connecting your wallet - the interface will show you what options are available.

## Governance Tokens

Governance tokens (Token ID 0) are the foundation of MarketDAO participation. They allow you to:
- Create and support proposals
- Claim voting tokens during elections
- Participate in DAO governance

### Acquiring Governance Tokens

There are two ways to acquire governance tokens:

1. **Direct Purchase:** On the Dashboard, enter the amount of tokens you wish to purchase and click "Purchase". The cost will be calculated based on the current token price.

2. **Secondary Markets:** Governance tokens can be bought and sold on supported ERC1155 marketplaces. Look for the "Buy/Sell on Rarible" link on the Dashboard for direct access.

> **Note:** If the token price is set to 0, direct purchases are disabled.

#### Token Vesting

Governance tokens acquired through direct purchase are subject to a vesting period:
- **Vesting Period:** Purchased tokens are locked for a configured number of blocks
- **Unlocking:** Tokens gradually become available for governance as they vest
- **Governance Rights:** Only vested (unlocked) tokens can be used to support proposals or claim voting tokens
- **Trading:** You can transfer governance tokens at any time, but the vesting schedule transfers with them
- **Automatic Cleanup:** Expired vesting schedules are automatically removed when transferring tokens
- **Schedule Consolidation:** Multiple purchases with the same unlock time are automatically merged
- **Schedule Limit:** Maximum 10 active vesting schedules per address (prevents DoS attacks)
- **Manual Cleanup:** You can call `cleanupMyVestingSchedules()` to remove expired schedules anytime
- **Dashboard Display:** The interface shows total, vested, and unvested balances separately so you can track your governance power

This vesting mechanism protects the DAO from hostile takeover attempts where an attacker might try to purchase a large number of tokens and immediately vote themselves control of the treasury.

**Note**: Initial token holders (those who received tokens when the DAO was created) are not subject to vesting restrictions.

### Checking Your Balance

Your governance token balance is displayed prominently on the Dashboard under "Your Token Balance".

## Creating Proposals

MarketDAO supports four types of proposals. To create a proposal:

1. Navigate to the Dashboard
2. Scroll to the "Create Proposal" section
3. Select the proposal type
4. Fill in the required information
5. Click the "Create" button for your chosen proposal type

### Resolution Proposals

Resolution proposals are text-only governance decisions.

Required information:
- **Description:** A clear explanation of what is being proposed

### Treasury Proposals

Treasury proposals transfer assets from the DAO treasury to a specified recipient.

Required information:
- **Description:** Purpose of the transfer
- **Recipient Address:** Wallet address that will receive the assets
- **Amount:** Quantity to transfer
- **Token Type:** ETH, ERC20, ERC721, or ERC1155
- **Token Address:** Contract address (for token types other than ETH)
- **Token ID:** Specific token ID (for ERC721 or ERC1155)

### Mint Proposals

Mint proposals create new governance tokens for a specified recipient.

Required information:
- **Description:** Purpose of the token minting
- **Recipient Address:** Wallet address that will receive the new tokens
- **Amount:** Number of governance tokens to mint

> **Note:** Mint proposals will only work if token minting is allowed in the DAO configuration.

### Parameter Proposals

Parameter proposals allow you to change DAO configuration parameters through democratic governance. This is a powerful feature that gives the community control over how the DAO operates.

Required information:
- **Description:** Justification for the parameter change
- **Parameter Type:** Select which parameter to modify
- **New Value:** The proposed new value for the selected parameter

**Available Parameter Types:**

1. **Support Threshold**
   - What it controls: Percentage of governance tokens needed to trigger an election
   - Valid range: 0.01% to 100% (in basis points: > 0 and <= 10000)
   - Example use: Lower to make elections easier to trigger, raise to require more community backing

2. **Quorum Percentage**
   - What it controls: Percentage participation needed for a valid election
   - Valid range: 1% to 100% (in basis points: >= 100 and <= 10000)
   - Example use: Lower for faster decisions, raise to require broader participation

3. **Max Proposal Age**
   - What it controls: How long a proposal can exist before expiring (in blocks)
   - Valid range: Must be > 0
   - Example use: Increase to give proposals more time to gather support

4. **Election Duration**
   - What it controls: Length of the voting period (in blocks)
   - Valid range: Must be > 0
   - Example use: Extend for more deliberation time, shorten for faster decisions

5. **Vesting Period**
   - What it controls: Vesting period for purchased governance tokens (in blocks)
   - Valid range: >= 0 (0 disables vesting)
   - Example use: Increase for more protection against hostile takeovers, decrease to allow faster participation

6. **Token Price**
   - What it controls: Price for direct token purchases (in wei)
   - Valid range: Must be > 0 (or 0 to disable direct sales)
   - Example use: Adjust based on DAO treasury needs or token value

7. **Flags**
   - What it controls: Boolean configuration options (bitfield)
   - Valid range: 0-7 (bits 0-2 only)
   - Bits:
     - Bit 0: Allow minting (can new governance tokens be minted)
     - Bit 1: Restrict purchases (limit to existing holders)
     - Bit 2: Mint to purchase (mint new tokens vs transfer from treasury)
   - Example use: Enable/disable features like token minting or purchase restrictions

**Important Notes:**
- Parameter proposals provide validation to prevent invalid configurations
- Changes take effect immediately when the proposal is executed
- The DAO name, treasury configuration, and purchase restrictions cannot be changed (set at deployment)

## Supporting Proposals

For a proposal to trigger an election, it needs to reach the support threshold. To support a proposal:

1. Navigate to the "Proposals" tab
2. Find the proposal you wish to support
3. Enter the amount of support to add (this is limited by your governance token holdings)
4. Click "Add Support"
5. Confirm the transaction in your wallet
6. Once the support threshold is reached, the proposal will automatically trigger an election

## Participating in Elections

When a proposal reaches the support threshold, an election is triggered. This is where MarketDAO's unique market-driven governance comes into play.

### Understanding the Election Process

The election process follows these steps:

1. **Claim Period:** When an election starts, governance token holders can claim voting tokens (Token ID corresponding to the proposal ID) equal to their vested governance token balance
2. **Trading Period:** During the election, voting tokens can be freely bought and sold
3. **Voting Period:** Voting tokens can be used to cast votes by sending them to YES or NO addresses
4. **Result Determination:** At the end of the election period, the proposal passes if the YES votes exceed NO votes and the quorum threshold is met

#### Claiming Voting Tokens

MarketDAO uses a "lazy minting" approach to reduce gas costs:

1. Navigate to the "Elections" tab to view active elections
2. For each election, you'll see a "Claim Voting Tokens" button if you haven't claimed yet
3. Click the button to claim your voting tokens (you'll receive tokens equal to your vested governance token balance)
4. Confirm the transaction in your wallet
5. Once claimed, you can vote with your tokens or trade them on supported marketplaces

**Why lazy minting?** Instead of automatically distributing voting tokens to all governance token holders when an election starts (which would cost significant gas), tokens are only minted when holders actively claim them. This means:
- Proposal creators don't pay massive gas fees to distribute tokens
- Only participants who actually want to vote pay gas to claim their tokens
- Non-participating token holders don't waste gas on tokens they won't use

### Voting on Proposals

To vote on an active proposal:

1. Navigate to the "Elections" tab
2. Find the active election you wish to vote on
3. Select "Vote YES" or "Vote NO"
4. Enter the number of voting tokens to use
5. Click "Cast Vote"
6. Confirm the transaction in your wallet

### Trading Voting Tokens

The ability to trade voting tokens is what makes MarketDAO unique. To trade voting tokens:

1. Navigate to the "Elections" tab
2. Find the election with voting tokens you want to trade
3. Click on "Trade Tokens"
4. You'll be redirected to the marketplace where you can list your tokens for sale or purchase tokens from others

> **Note:** You can also use other compatible marketplaces that support ERC1155 tokens.

## Viewing Proposal History

To view past proposals and elections:

1. Navigate to the "History" tab
2. Browse through the list of past proposals
3. Click on any proposal to view details including support levels, voting results, and transaction history

## FAQ

**Q: What happens to my purchased governance tokens during the vesting period?**
A: During the vesting period, you own the tokens and can transfer them, but you cannot use them for governance (supporting proposals or claiming voting tokens) until they vest. The vesting schedule transfers with the tokens if you sell them.

**Q: Do I have to claim voting tokens for every election?**
A: Yes, voting tokens must be claimed separately for each election. You can only claim once per election, and the amount you receive equals your vested governance token balance at the time of claim.

**Q: What if I don't claim my voting tokens?**
A: If you don't claim your voting tokens, you won't be able to vote in that election. However, your vested governance tokens are still counted in quorum and majority calculations, so unclaimed tokens don't affect the validity of the election results.

**Q: Can I get my governance tokens back after voting?**
A: No, once governance tokens are used to create or support proposals, they remain locked until the proposal process completes.

**Q: What happens if I sell my voting tokens during an election?**
A: When you sell voting tokens, you're effectively transferring your voting power to the buyer. The buyer can then use those tokens to vote or sell them to others.

**Q: Why would someone buy my voting tokens?**
A: People may buy voting tokens if they feel strongly about the outcome of a proposal. This creates a market-based assessment of the proposal's importance to different stakeholders.

**Q: What happens if the quorum isn't reached?**
A: If the quorum threshold isn't met by the end of the election period, the proposal fails regardless of the YES/NO vote balance.

**Q: Can proposals be canceled?**
A: Once created, proposals cannot be canceled. They will either fail to reach the support threshold and expire, or they will proceed to an election and be decided by voting.

**Q: What happens if my join request is rejected?**
A: If your join request is rejected, you do not receive any governance tokens and cannot submit another join request from the same wallet address. However, you can still observe the DAO's activities and proposals.

**Q: Can I submit multiple join requests?**
A: No, you can only submit one join request per wallet address. This prevents spam and ensures that members take join requests seriously. If you need to try again, you would need to use a different wallet address.

**Q: How do I know if a DAO has purchase restrictions enabled?**
A: Connect your wallet to the DAO interface. If the DAO has purchase restrictions and you don't hold tokens, you'll see the join request interface instead of a direct purchase option. DAOs in open mode will show direct purchase options to everyone.

**Q: What are vesting schedules and why do they matter?**
A: Vesting schedules track when your purchased tokens unlock for governance use. You can have up to 10 active schedules, and the system automatically cleans up expired ones. This prevents attacks where someone might try to accumulate many small purchases to overflow the system.

**Q: Can I see how many vested vs. unvested tokens I have?**
A: Yes, the dashboard displays your total, vested, and unvested balances separately so you always know how much governance power you currently have available.

**Q: What parameters can be changed through parameter proposals?**
A: Seven parameters can be changed: support threshold, quorum percentage, max proposal age, election duration, vesting period, token price, and flags (boolean options). However, the DAO name, treasury configuration, and purchase restrictions are permanent and cannot be changed after deployment.

**Q: Can parameter changes be reversed?**
A: Yes, any parameter change can be reversed by creating another parameter proposal to change it back. The community always has the power to adjust parameters through democratic voting.

**Q: What happens if I submit an invalid parameter value?**
A: Parameter proposals have built-in validation. For example, you cannot set quorum below 1% or support threshold above 100%. Invalid values will be rejected when you try to create the proposal.

**Q: How do I change a flag bit?**
A: Flags are stored as a bitfield (0-7). To change a specific flag, you need to calculate the new bitfield value with that flag toggled. The frontend provides checkboxes to make this easier. For example, if current flags are 3 (bits 0 and 1 set) and you want to also enable bit 2, submit 7 (all three bits set).