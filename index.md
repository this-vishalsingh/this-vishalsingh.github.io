---
layout: default
title: Josselin Feist
---

<style>
.nav-tabs {
  display: flex;
  margin-bottom: 1.5rem;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-family: inherit;
  font-size: 0.95em;
  overflow: hidden;
  width: fit-content;
}

.nav-tabs a {
  padding: 8px 16px;
  text-decoration: none;
  color: #0366d6;
  background: #f8f8f8;
  border-right: 1px solid #ccc;
  flex: 1;
  text-align: center;
}

.nav-tabs a:last-child {
  border-right: none;
}

.nav-tabs a.active {
  background: white;
  font-weight: bold;
}
</style>

<div class="nav-tabs">
  <a href="/" class="active">Home</a>
  <a href="/blog">Blog</a>
  <a href="/talks/">Talks</a>
  <a href="/portfolio/">Portfolio</a>
  <a href="/about/">About</a>
</div>


---

*Former Trail of Bits Engineering Director – Author of Slither*

---

## Profile

I'm a seasoned information security expert with over a decade of experience, including **8+ years focused on blockchain security**. Until 2025, I was the Engineering Director of the Blockchain team at Trail of Bits, where I led reviews of some of the most critical systems in the space. I've recently transitioned to independent consulting and am now **available for private blockchain security reviews**.

I specialize in securing both cutting-edge DeFi applications and the core blockchain components.

- **Deep DeFi Expertise:** I've reviewed AMMs, stablecoins, lending platforms, vaults, and liquid (re)staking systems.
- **Core Blockchain Protocols:** I have deep experience auditing L1 and L2 systems, rollups, bridges, consensus, and messaging layers.
- **Multi-Language Security Expertise:** I'm highly proficient in **Solidity**, **Yul**, **Rust**, and **Go**, with years of hands-on auditing and development experience.
- **Alt-Chain Experience:** I've also worked on protocols like Move, Solana, Cosmos, Polkadot, and Algorand, adapting my methods to their unique architectures.

## Notable Security Review Outcomes

Over the years, I've had the opportunity to lead or contribute to security reviews that uncovered critical issues and helped shape best practices across the blockchain space:

- **Upgradeability Anti-Patterns:** I wrote a series of blog posts on smart contract upgrade risks, including [*“Contract upgrade anti-patterns”*](https://blog.trailofbits.com/2018/09/05/contract-upgrade-anti-patterns/) and [*“Good idea, bad design: How the Diamond standard falls short”*](https://blog.trailofbits.com/2020/10/30/good-idea-bad-design-how-the-diamond-standard-falls-short/). These articles aim to help developers avoid common pitfalls when designing upgradable smart contracts.
- **Critical Aave Upgradeability Bug:** I discovered a severe vulnerability in [Aave's upgradeable proxy contracts](https://blog.trailofbits.com/2020/12/16/breaking-aave-upgradeability/) that could have “broken” Aave and affected multiple integrated DeFi protocols. This issue was the result of extensive research I conducted into upgrade mechanisms and proxy design.
- **Arbitrum Nitro L2 Security Review:** I led the audit of [**Arbitrum Nitro**](https://docs.arbitrum.io/assets/files/2022_03_14_trail_of_bits_security_audit_nitro_1_of_2-d777111730bd602222978f7d98713d40.pdf), a complex Ethereum Layer-2 rollup. The review involved deep analysis of the L2 state transition function, fraud proofs, and bridging logic.
- **Tezos Message-Passing Vulnerability:** I identified and publicly documented critical flaws in Tezos's [message-passing model](https://forum.tezosagora.org/t/smart-contract-vulnerabilities-due-to-tezos-message-passing-architecture/2045). This included issues like callback authorization bypass and call injection—bugs that hadn't been previously identified in the ecosystem.
- **Balancer V2 Audit:** As the lead reviewer of Balancer's V2 codebase, I performed an [in-depth analysis of numerical precision issues](https://github.com/trailofbits/publications/blob/master/reviews/2021-04-balancer-balancerv2-securityreview.pdf) and authored **Appendix H**, which covered rounding error impacts. This work directly influenced safer arithmetic practices within the protocol.

*See the [Portfolio](/portfolio/) tab for a full list of public security reviews.*

## Open Source & Research

Beyond audits, I've spent years building tools that help the broader security community and advancing research in program analysis.

- **Slither:** I created [**Slither**](https://github.com/crytic/slither), the leading static analysis framework for Ethereum smart contracts. It's widely used to detect vulnerabilities and weaknesses in Solidity and Vyper code, helping auditors, developers, and protocol teams improve their security posture.
- **Security Tools:** I've developed other open-source tools such as [*Tealer*](https://github.com/crytic/tealer), a static analyzer for Algorand's TEAL smart contracts, and [*RoundMe*](https://github.com/crytic/roundme), a toolkit for identifying rounding errors in smart contract arithmetic.
- **Academic Background:** I hold a **Ph.D. in program analysis for security**, which has shaped my systematic approach to auditing and building security tools grounded in theory and proven techniques.

## Philosophy & Approach

**"Securing systems at the edge of innovation—driven by curiosity, grounded in rigor".**

What drives me is the challenge of uncovering non-obvious flaws in complex systems. I'm energized by deep technical puzzles, subtle inconsistencies, and the opportunity to explore areas that have not been thoroughly examined before.

*See the [About](/about/) to read more about my approach.*

---

## What I Work On

I am most effective when working with systems that are new, unconventional, or pushing the boundaries of what has been done before.

- **Finding deep, design-level bugs** that aren't caught by standard checklists or automated tools.
- **Auditing novel, high-risk protocols** where there is little precedent or existing research.
- **Helping teams mature their security practices**, from foundational architecture to ongoing advisory. I created the [Blockchain Security Maturity Model](https://blog.trailofbits.com/2023/07/14/evaluating-blockchain-security-maturity/) to guide teams in building long-term resilience.

I don't just deliver reports. I help teams build stronger systems.

## Contact

Interested in a **private security audit** or have a question? Feel free to reach out:

- **Email:** [josselin@seceureka.com](mailto:josselin@seceureka.com)  
- **Telegram:** [@montyly](https://t.me/montyly)  
- **Twitter:** [@montyly](https://x.com/Montyly)
- **Book a call:** [30 min call](https://calendar.app.google/uyV1CaY5pLF5z7baA) 

---
*Content last updated January 2026.*
