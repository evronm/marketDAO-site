---
layout: page
title: MarketDAO User Guide
permalink: /docs/user-guide/
---

## Table of Contents

1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
   - [Deploying a MarketDAO Instance](#deploying-a-marketdao-instance)
   - [Understanding the Dashboard](#understanding-the-dashboard)
3. [Join Request System (For Non-Members)](#join-request-system-for-non-members)
   - [How to Submit a Join Request](#how-to-submit-a-join-request)
   - [Join Request Process](#join-request-process)
4. [Governance Tokens](#governance-tokens)
   - [Acquiring Governance Tokens](#acquiring-governance-tokens)
   - [Checking Your Balance](#checking-your-balance)
   - [Token Locking](#token-locking)
5. [Creating Proposals](#creating-proposals)
   - [Resolution Proposals](#resolution-proposals)
   - [Treasury Proposals](#treasury-proposals)
   - [Mint Proposals](#mint-proposals)
   - [Parameter Proposals](#parameter-proposals)
   - [Distribution Proposals](#distribution-proposals)
6. [Supporting Proposals](#supporting-proposals)
7. [Participating in Elections](#participating-in-elections)
   - [Understanding the Election Process](#understanding-the-election-process)
   - [Voting on Proposals](#voting-on-proposals)
   - [Trading Voting Tokens](#trading-voting-tokens)
8. [Releasing Locked Tokens](#releasing-locked-tokens)
9. [Claiming Distributions](#claiming-distributions)
10. [Viewing Proposal History](#viewing-proposal-history)
11. [FAQ](#faq)

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
2. **Access Join Request**: You'll see a "Request to Join" option instead of the standard proposal creation interface
3. **Write Your Introduction**: Describe yourself and why you want to join the DAO
4. **Submit Request**: Your join request is created as a special mint proposal for 1 governance token

### Join Request Process

1. **Support Phase**: Existing members can add support to your join request
2. **Election**: If the support threshold is reached, an election is triggered
3. **Voting**: Members vote YES or NO on your admission
4. **Resolution**: If approved, you receive 1 governance token and become a member

## Governance Tokens

### Acquiring Governance Tokens

There are several ways to acquire governance tokens:

1. **Initial Distribution**: Tokens allocated at DAO creation
2. **Direct Purchase**: Buy tokens at the configured price (if enabled)
3. **Join Request**: Submit a join request to be voted in (if purchases are restricted)
4. **Mint Proposal**: Receive tokens through an approved mint proposal

### Checking Your Balance

The dashboard displays several balance types:

- **Total Balance**: All governance tokens you hold
- **Vested Balance**: Tokens available for governance (purchased tokens vest over time)
- **Transferable Balance**: Tokens you can transfer (vested minus locked)
- **Locked Balance**: Tokens locked for active proposals or distributions

### Token Locking

To prevent double-counting attacks, governance tokens are automatically locked when you:

- **Add Support to a Proposal**: Tokens are locked until the proposal resolves
- **Claim Voting Tokens**: Your vested balance is locked during the election
- **Register for a Distribution**: Tokens are locked until you claim or the distribution ends

**Important**: Locked tokens cannot be transferred, but they still count toward your governance power for the actions that locked them.

## Creating Proposals

Only addresses with vested governance tokens can create proposals (except join requests). Each proposal type serves a different purpose:

### Resolution Proposals

Text-only governance decisions for signaling and community direction.

**Use for:**
- Community guidelines or policies
- Strategic direction decisions
- Non-binding statements

### Treasury Proposals

Transfer assets from the DAO treasury to a recipient.

**Parameters:**
- Recipient address
- Amount to transfer
- Asset type (ETH, ERC20, ERC721, or ERC1155)

**Note:** Treasury funds are locked when the election triggers, not at proposal creation.

### Mint Proposals

Create new governance tokens (requires minting to be enabled).

**Parameters:**
- Recipient address
- Amount of tokens to mint

**Use for:**
- Rewarding contributors
- Adding new members (alternative to join requests)
- Expanding governance participation

### Parameter Proposals

Modify DAO configuration through governance.

**Changeable Parameters:**
- Support Threshold (0-100% in basis points)
- Quorum Percentage (1-100% in basis points)
- Max Proposal Age (in blocks)
- Election Duration (in blocks)
- Vesting Period (in blocks, 0 disables)
- Token Price (in wei, 0 disables direct sales)
- Flags (allow minting, restrict purchases, mint to purchase)

### Distribution Proposals

Proportionally distribute treasury assets to all governance token holders.

**Parameters:**
- Asset to distribute (ETH, ERC20, or ERC1155)
- Amount per governance token (target)

**Note:** Actual payouts are calculated pro-rata based on total registered shares vs actual pool balance.

## Supporting Proposals

Before a proposal goes to election, it needs to gather support from governance token holders.

1. **View Proposal**: Navigate to the Proposals tab and select a proposal
2. **Add Support**: Enter the amount of tokens you want to use for support
3. **Confirm**: Submit the transaction

**Token Locking**: When you add support, those tokens are **locked** until the proposal resolves. You can remove support before the election triggers to unlock your tokens.

**Threshold**: Once total support reaches the configured threshold (e.g., 20% of vested supply), an election is automatically triggered.

## Participating in Elections

### Understanding the Election Process

1. **Election Triggers**: When support threshold is reached
2. **Snapshot**: Your voting power is frozen at election start
3. **Claim Tokens**: You must claim your voting tokens to participate
4. **Vote or Trade**: Use voting tokens to vote or trade them
5. **Resolution**: Election ends after the duration (or early if majority reached)

### Voting on Proposals

1. **Claim Voting Tokens**: Click "Claim Voting Tokens" to receive tokens equal to your vested governance balance
2. **Cast Your Vote**: Transfer voting tokens to the YES or NO address
3. **Full or Partial**: You can vote with all or some of your voting tokens

**Token Locking**: When you claim voting tokens, your governance tokens are **locked** until the proposal resolves.

### Trading Voting Tokens

Voting tokens are standard ERC1155 tokens and can be:

- Transferred to other addresses
- Traded on NFT marketplaces
- Exchanged peer-to-peer

**Strategy**: If you feel strongly about a proposal, you can acquire additional voting tokens. If you're indifferent, you can sell your voting tokens to someone who cares more.

**Restriction**: Voting tokens cannot be transferred to vote addresses after the election ends.

## Releasing Locked Tokens

After a proposal resolves (executed, failed, or expired), you need to release your locked tokens:

1. **Navigate to Proposal**: Find the resolved proposal
2. **Release Locks**: Click "Release Locks" to unlock your tokens
3. **Confirm**: Submit the transaction

**Why Manual Release?** This design shifts gas costs to users who need to transfer tokens, rather than making the proposal creator pay for everyone during execution.

**Distribution Locks**: For distributions, you can release locks by either:
- Claiming your distribution (automatically releases lock)
- Calling "Release Lock" after the distribution ends (if you don't want to claim)

## Claiming Distributions

If you registered for a distribution proposal that passed:

1. **Wait for Execution**: The proposal must be executed first
2. **Navigate to Distribution**: Find the distribution in your active claims
3. **Claim**: Click "Claim" to receive your share
4. **Receive Assets**: Pro-rata share is transferred to your wallet

**Pro-Rata Calculation**: Your share = (your registered tokens / total registered tokens) × actual pool balance

**Note**: The "amount per token" shown is a target. Actual amounts depend on total registrations and pool funding.

## Viewing Proposal History

The History tab shows all past proposals with their outcomes:

- **Executed**: Proposal passed and was executed
- **Failed**: Proposal didn't pass or quorum wasn't met
- **Expired**: Proposal didn't reach support threshold in time

## FAQ

### Why are my tokens locked?

Tokens are locked when you:
- Add support to an active proposal
- Claim voting tokens during an election
- Register for a distribution

This prevents the same tokens from being used multiple times (double-counting attacks).

### How do I unlock my tokens?

After the proposal resolves, navigate to it and click "Release Locks". This unlocks your tokens for future use.

### Why can't I transfer my tokens?

Check if you have locked tokens. Your transferable balance = vested balance - locked balance. Release locks from resolved proposals to increase your transferable balance.

### What happens if I sell my governance tokens after adding support?

Your support amount remains recorded, but you won't be able to vote with tokens you no longer hold. Support only determines if an election triggers—it doesn't affect voting outcomes.

### Can I vote after selling some governance tokens?

Yes, but only with the vested balance you had when you claimed voting tokens. Voting power is snapshotted at election start.

### What if a proposal expires?

If a proposal doesn't reach the support threshold before `maxProposalAge` blocks, it expires. You can release your locked tokens from expired proposals.

### How does early termination work?

If one side receives more than 50% of all possible votes before the election ends, the election terminates early. This prevents unnecessary waiting when the outcome is already determined.

### What's the difference between vested and unvested tokens?

Purchased tokens vest over a configurable period. Unvested tokens count toward your balance but cannot be used for governance (supporting, voting, or transferring). This prevents governance attacks from flash purchases.

### Can I participate in multiple proposals simultaneously?

Yes! Your locked tokens accumulate across all active proposals. For example, if you support Proposal A with 30 tokens and Proposal B with 20 tokens, 50 tokens are locked total.

### What is the quorum requirement?

Quorum is the minimum percentage of vested supply that must vote (YES + NO) for an election to be valid. If quorum isn't met, the proposal fails regardless of YES/NO ratio.

### How are distributions calculated?

Distribution uses pro-rata calculation: your share = (your registered tokens / total registered tokens) × actual pool balance. This ensures all registered users can claim even if the pool has less than expected.
