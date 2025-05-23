---
layout: default
title: Market-Driven Governance
---

<section class="section" id="concept">
    <h2>Core Concept</h2>
    <p>
        Unlike traditional DAOs where voting power is static, MarketDAO introduces a revolutionary approach:
        <strong>tradable voting tokens for each election</strong>.
    </p>
    <p>
        This creates a dynamic governance system where voters can:
    </p>
    <ul>
        <li>Buy more voting power if they feel strongly about an issue</li>
        <li>Sell their voting power if others value it more</li>
        <li>Speculate on election outcomes through voting token markets</li>
    </ul>
    <p>
        The result is a governance framework that more accurately captures the true strength of preferences 
        and creates efficient markets around governance decisions.
    </p>
</section>

<section class="section" id="features">
    <h2>Key Features</h2>
    <div class="feature-grid">
        <div class="feature-card">
            <h3>Saleable Voting Rights</h3>
            <p>Voting tokens are freely transferable during elections, allowing market forces to influence governance outcomes.</p>
        </div>
        <div class="feature-card">
            <h3>Multiple Proposal Types</h3>
            <p>Create resolutions, treasury transfers, governance token minting, and token price update proposals.</p>
        </div>
        <div class="feature-card">
            <h3>Flexible Support Thresholds</h3>
            <p>Customizable support thresholds and quorum percentages for different governance needs.</p>
        </div>
        <div class="feature-card">
            <h3>Treasury Management</h3>
            <p>Support for ETH, ERC20, ERC721, and ERC1155 assets in the DAO treasury.</p>
        </div>
        <div class="feature-card">
            <h3>Early Termination</h3>
            <p>Elections can end early when a clear majority is reached, improving efficiency.</p>
        </div>
        <div class="feature-card">
            <h3>ERC1155 Foundation</h3>
            <p>Built on the ERC1155 standard for flexible token management, with token ID 0 reserved for governance tokens.</p>
        </div>
    </div>
</section>

<section class="section" id="how-it-works">
    <h2>How It Works</h2>
    <ol>
        <li>
            <h3>Create a Proposal</h3>
            <p>Governance token holders can submit various types of proposals.</p>
        </li>
        <li>
            <h3>Support Phase</h3>
            <p>Proposals need to reach a support threshold before triggering an election.</p>
        </li>
        <li>
            <h3>Election</h3>
            <p>When triggered, voting tokens are distributed 1:1 to all governance token holders.</p>
        </li>
        <li>
            <h3>Trading Period</h3>
            <p>During elections, voting tokens can be freely bought and sold.</p>
        </li>
        <li>
            <h3>Voting</h3>
            <p>Cast votes by sending voting tokens to YES/NO addresses.</p>
        </li>
        <li>
            <h3>Execution</h3>
            <p>If the proposal passes and meets quorum, it is automatically executed.</p>
        </li>
    </ol>
</section>

<section class="section" id="implementation">
    <h2>Implementation Details</h2>
    <p>
        MarketDAO is built on Solidity with a focus on security and flexibility:
    </p>
    <ul>
        <li>Built on OpenZeppelin's ERC1155 implementation</li>
        <li>Proposal lifecycle with support thresholds and voting periods</li>
        <li>Inherits from ReentrancyGuard for security against reentrancy attacks</li>
        <li>Separate proposal types for different governance actions</li>
        <li>Factory pattern for easy proposal creation</li>
    </ul>
    
    <h3>Architecture</h3>
    <p>
        The system consists of three main components:
    </p>
    <ul>
        <li><strong>MarketDAO</strong>: The main contract that manages tokens, proposals, and voting</li>
        <li><strong>Proposal</strong>: Base class for all proposal types with the proposal lifecycle</li>
        <li><strong>ProposalFactory</strong>: Factory contract for creating different types of proposals</li>
    </ul>
</section>

<section class="section" id="configuration">
    <h2>Configuration Parameters</h2>
    <p>
        When creating a new DAO, you can configure:
    </p>
    <ul>
        <li><strong>Name</strong>: The name of the DAO</li>
        <li><strong>Support Threshold</strong>: Percentage of tokens needed to trigger an election</li>
        <li><strong>Quorum Percentage</strong>: Percentage of tokens needed for a valid election</li>
        <li><strong>Maximum Proposal Age</strong>: Time before a proposal expires if it doesn't trigger an election</li>
        <li><strong>Election Duration</strong>: Length of the voting period in blocks</li>
        <li><strong>Treasury Configuration</strong>: What asset types the treasury can hold (ETH, ERC20, ERC721, ERC1155)</li>
        <li><strong>Allow Minting</strong>: Whether new governance tokens can be minted through proposals</li>
        <li><strong>Token Price</strong>: Initial price for direct token purchases (0 disables direct sales)</li>
        <li><strong>Initial Token Distribution</strong>: Starting allocation of governance tokens</li>
    </ul>
</section>

<section class="section" id="deployment">
    <h2>Try It Out</h2>
    <p>
        MarketDAO is deployed on the Polygon Amoy testnet and ready for you to experiment with:
    </p>
    <ul>
        <li><strong>Live Demo</strong>: <a href="https://evronm.github.io/marketDAO/index.html" target="_blank">https://evronm.github.io/marketDAO/index.html</a></li>
        <li><strong>DAO Contract</strong>: 0xf188d689d78b58b9d3e1a841a9b9afb8f92ddf55</li>
        <li><strong>Factory Contract</strong>: 0xc609fa60239116ecee53af12f733eb0214d7b1ad</li>
    </ul>
    <p>
        You'll need a wallet connected to the Polygon Amoy testnet to interact with the demo.
    </p>
</section>

<section class="section">
    <h2>Future Possibilities</h2>
    <p>
        MarketDAO is just getting started, with plans for:
    </p>
    <ul>
        <li>Resolution enhancements: Expiring resolutions, cancellation proposals</li>
        <li>Multiple choice proposals beyond binary YES/NO</li>
        <li>Variable election lengths based on proposal type</li>
        <li>Staking mechanisms to prevent proposal spam</li>
        <li>Enhanced market integrations for voting token exchanges</li>
    </ul>
</section>