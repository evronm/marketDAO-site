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
3. [Governance Tokens](#governance-tokens)
   - [Acquiring Governance Tokens](#acquiring-governance-tokens)
   - [Checking Your Balance](#checking-your-balance)
4. [Creating Proposals](#creating-proposals)
   - [Resolution Proposals](#resolution-proposals)
   - [Treasury Proposals](#treasury-proposals)
   - [Mint Proposals](#mint-proposals)
   - [Token Price Proposals](#token-price-proposals)
5. [Supporting Proposals](#supporting-proposals)
6. [Participating in Elections](#participating-in-elections)
   - [Understanding the Election Process](#understanding-the-election-process)
   - [Voting on Proposals](#voting-on-proposals)
   - [Trading Voting Tokens](#trading-voting-tokens)
7. [Viewing Proposal History](#viewing-proposal-history)
8. [FAQ](#faq)

## Introduction

MarketDAO is a governance framework that brings market forces to group decisions. The key innovation is that **voting rights can be bought and sold during elections**, allowing market forces to influence governance outcomes. This user guide will walk you through all aspects of participating in MarketDAO governance.

## Getting Started

### Connecting Your Wallet

1. Visit the MarketDAO application at [https://evronm.github.io/marketDAO/index.html](https://evronm.github.io/marketDAO/index.html)
2. Click the "Connect Wallet" button
3. If prompted, select your wallet provider (MetaMask, etc.)
4. Approve the connection request in your wallet
5. Once connected, your wallet address will be displayed and you'll gain access to the full functionality of the platform

> **Note:** MarketDAO is currently deployed on the Polygon Amoy testnet. Make sure your wallet is configured for this network.

### Understanding the Dashboard

After connecting your wallet, you'll see the Dashboard, which displays:

- **DAO Information** - Name, your token balance, total token supply, token price, etc.
- **Token Purchase** - Interface to buy governance tokens
- **Navigation Tabs** - Access to Proposals, Elections, and History sections

## Governance Tokens

Governance tokens (Token ID 0) are the foundation of MarketDAO participation. They allow you to:
- Create and support proposals
- Receive voting tokens during elections
- Participate in DAO governance

### Acquiring Governance Tokens

There are two ways to acquire governance tokens:

1. **Direct Purchase:** On the Dashboard, enter the amount of tokens you wish to purchase and click "Purchase". The cost will be calculated based on the current token price.

2. **Secondary Markets:** Governance tokens can be bought and sold on supported marketplaces. Look for the "Buy/Sell on Rarible" link on the Dashboard for direct access.

> **Note:** If the token price is set to 0, direct purchases are disabled.

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

### Token Price Proposals

Token price proposals change the price of governance tokens for direct purchase.

Required information:
- **Description:** Justification for the price change
- **New Price (in ETH):** The proposed token price (set to 0 to disable direct sales)

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

1. **Token Distribution:** When an election starts, every governance token holder receives an equal number of voting tokens (Token ID corresponding to the proposal ID)
2. **Trading Period:** During the election, voting tokens can be freely bought and sold
3. **Voting Period:** Voting tokens can be used to cast votes by sending them to YES or NO addresses
4. **Result Determination:** At the end of the election period, the proposal passes if the YES votes exceed NO votes and the quorum threshold is met

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