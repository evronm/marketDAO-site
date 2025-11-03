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
            <h3>Governance Token Vesting</h3>
            <p>Purchased governance tokens are subject to a vesting period with automatic cleanup and consolidation, protecting the DAO from hostile takeover attempts.</p>
        </div>
        <div class="feature-card">
            <h3>Lazy Vote Token Distribution</h3>
            <p>Voting tokens are minted on-demand as participants claim them, reducing gas costs for proposal creators.</p>
        </div>
        <div class="feature-card">
            <h3>Purchase Restrictions (Optional)</h3>
            <p>Optionally limit token purchases to existing holders, requiring new members to be approved via governance proposals.</p>
        </div>
        <div class="feature-card">
            <h3>Join Request System</h3>
            <p>Non-holders can submit join requests to become members, which are voted on by existing token holders.</p>
        </div>
        <div class="feature-card">
            <h3>Snapshot-Based Voting Power</h3>
            <p>Unlimited scalability with O(1) snapshot creation, supporting 10,000+ holders without gas limit concerns.</p>
        </div>
        <div class="feature-card">
            <h3>Multiple Proposal Types</h3>
            <p>Create resolutions, treasury transfers, governance token minting, and parameter change proposals.</p>
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
        <div class="feature-card">
            <h3>Configurable Parameters via Governance</h3>
            <p>All major governance parameters can be modified through democratic voting, including support thresholds, quorum requirements, election duration, vesting periods, and token prices.</p>
        </div>
    </div>
</section>

<section class="section" id="how-it-works">
    <h2>How It Works</h2>

    <h3>For New Members (Join Request):</h3>
    <ol>
        <li>
            <h3>Connect Wallet</h3>
            <p>Non-holders connect their wallet to the DAO interface.</p>
        </li>
        <li>
            <h3>Submit Join Request</h3>
            <p>Provide a description introducing yourself and why you want to join.</p>
        </li>
        <li>
            <h3>Wait for Support</h3>
            <p>Existing members review your request and add support if they approve.</p>
        </li>
        <li>
            <h3>Election</h3>
            <p>Once support threshold is met, members vote on your admission.</p>
        </li>
        <li>
            <h3>Admission</h3>
            <p>If approved, you receive 1 governance token and full DAO access.</p>
        </li>
    </ol>

    <h3>For Token Holders (Standard Proposals):</h3>
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
            <p>When triggered, governance token holders can claim voting tokens 1:1 with their vested governance token balance.</p>
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
        <li><strong>Support Threshold</strong>: Percentage (in basis points) needed to trigger an election</li>
        <li><strong>Quorum Percentage</strong>: Percentage (in basis points) needed for a valid election</li>
        <li><strong>Maximum Proposal Age</strong>: Time before a proposal expires if it doesn't trigger an election (in blocks)</li>
        <li><strong>Election Duration</strong>: Length of the voting period in blocks</li>
        <li><strong>Flags</strong>: Bitfield for boolean options
            <ul>
                <li>Bit 0: Allow minting (whether new governance tokens can be minted via proposals)</li>
                <li>Bit 1: Restrict purchases (whether token purchases are limited to existing holders)</li>
                <li>Bit 2: Mint to purchase (whether to mint new tokens or transfer from treasury when selling tokens)</li>
            </ul>
        </li>
        <li><strong>Token Price</strong>: Initial price for direct token purchases in wei (0 disables direct sales)</li>
        <li><strong>Vesting Period</strong>: Vesting period for purchased governance tokens in blocks (0 disables vesting)</li>
        <li><strong>Treasury Configuration</strong>: What asset types the treasury can hold (ETH, ERC20, ERC721, ERC1155)</li>
        <li><strong>Initial Token Distribution</strong>: Starting allocation of governance tokens (addresses and amounts)</li>
    </ul>
    <p>
        <strong>Note on Basis Points:</strong> All percentage parameters use basis points for precision (10000 = 100%, 5100 = 51%, 2000 = 20%, 250 = 2.5%), allowing for 0.01% precision.
    </p>
</section>

<section class="section" id="deployment">
    <h2>Get Started</h2>
    <p>
        MarketDAO is open source and ready for deployment:
    </p>
    <ul>
        <li><strong>Repository</strong>: <a href="https://github.com/evronm/marketDAO" target="_blank">https://github.com/evronm/marketDAO</a></li>
        <li><strong>Contracts</strong>: See the <code>src/</code> directory for all Solidity contracts</li>
        <li><strong>Frontend</strong>: Included in the repository and compatible with the latest contracts</li>
    </ul>
    <p>
        Clone the repository and follow the deployment instructions in the <a href="{{ site.baseurl }}/docs/technical-reference/#deployment-guide">Technical Reference</a> to deploy your own instance on any EVM-compatible network.
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