---
layout: default
title: MarketDAO - Market-Driven Governance
---

<section class="section" id="concept">
    <h2>Core Concept</h2>
    <p>
        MarketDAO is a governance framework that brings market forces to bear on group decisions.
        Unlike traditional DAOs where voting power is static, MarketDAO introduces tradable voting tokens for each election.
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
            <p>Create resolutions, treasury transfers, governance token minting, parameter changes, and distribution proposals.</p>
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
            <p>Proposals need to reach a support threshold to trigger an election. Tokens used for support are locked to prevent double-counting.</p>
        </li>
        <li>
            <h3>Election Period</h3>
            <p>Once triggered, an election begins. Each governance token holder can claim voting tokens equal to their vested balance.</p>
        </li>
        <li>
            <h3>Vote or Trade</h3>
            <p>Voting tokens can be used to vote (transfer to YES/NO addresses) or traded with others who feel more strongly.</p>
        </li>
        <li>
            <h3>Resolution</h3>
            <p>After the election period (or early if a clear majority is reached), the proposal is executed or rejected based on results.</p>
        </li>
        <li>
            <h3>Unlock Tokens</h3>
            <p>After resolution, users can release their locked governance tokens for future proposals.</p>
        </li>
    </ol>
</section>

<section class="section" id="implementation">
    <h2>Implementation</h2>
    <p>
        MarketDAO is built on Solidity ^0.8.20 with OpenZeppelin libraries, using the ERC1155 multi-token standard for both governance and voting tokens.
    </p>
    <h3>Security Audit</h3>
    <p>
        MarketDAO has been <strong>audited by <a href="https://hashlock.com/audits/marketdao" target="_blank" rel="noopener">Hashlock Pty Ltd</a></strong> (January 2026). All HIGH severity issues have been resolved.
    </p>
    <p style="text-align: center; margin: 20px 0;">
        <a href="https://hashlock.com/audits/marketdao" target="_blank" rel="noopener">
            <img src="{{ site.baseurl }}/assets/images/hashlock-badge.png" alt="Audited by Hashlock" style="max-width: 180px;">
        </a>
    </p>
    <p style="text-align: center;">
        <a href="https://hashlock.com/audits/marketdao" target="_blank" rel="noopener" class="cta-button">View Full Audit Report</a>
    </p>
    <h3>Security Features</h3>
    <div class="feature-grid">
        <div class="feature-card">
            <h3>Governance Token Locking</h3>
            <p>Tokens are locked when used for proposal support or voting claims, preventing double-counting attacks.</p>
        </div>
        <div class="feature-card">
            <h3>Distribution Token Locking</h3>
            <p>Tokens are locked when registering for distributions, preventing double-claim attacks.</p>
        </div>
        <div class="feature-card">
            <h3>Operator Voting Restrictions</h3>
            <p>Election-ended checks apply to all transfers, not just direct transfers.</p>
        </div>
        <div class="feature-card">
            <h3>Pro-Rata Distributions</h3>
            <p>Distribution claims use proportional calculations to prevent pool exhaustion.</p>
        </div>
        <div class="feature-card">
            <h3>Reentrancy Protection</h3>
            <p>ReentrancyGuard on transfer functions prevents reentrancy during vote transfers.</p>
        </div>
        <div class="feature-card">
            <h3>O(1) Scalability</h3>
            <p>Snapshot-based voting enables 10,000+ participants with constant gas costs.</p>
        </div>
    </div>
    <p>
        See the <a href="{{ site.baseurl }}/docs/technical-reference/#security--scalability">Technical Reference</a> for full security details.
    </p>
</section>

<section class="section" id="configuration">
    <h2>Configuration</h2>
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
    <h2>Try It</h2>
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
