---
layout: default
active: home
description: "Multi-Ecosystem Security Engineer & Cryptography Researcher. Specializing in DeFi Smart Contracts, Infrastructure, and ZK Systems."
---

Multi-Ecosystem Security Engineer & Cryptography Researcher. I audit the most complex surfaces in Web3—from **Solidity/Rust** smart contracts and **L1/L2 infrastructure**, to bleeding-edge **zkVMs, FHE, and ZK circuits (Cairo/Noir)**.

<div class="cta-buttons">
  <a href="https://calendar.app.google/T47tbEv9ta2D1gws5" target="_blank" rel="noopener noreferrer" class="cta-btn cta-btn-primary">Request an Audit</a>
</div>

---

## Profile

Operating under the research banner **[ZippelLabs](https://zippellabs.github.io/)**, I specialize in analyzing intricate architectures that resist easy explanations and bring the most value when evaluating unconventional, frontier systems.

My deep background in advanced cryptography (ZK/FHE) allows me to find edge-case architectural flaws in core DeFi protocols, while my extensive smart contract experience grounds my theoretical infrastructure reviews with practical exploit mechanics.

- **Finding deep, design-level bugs** that standard checklists or automated tools miss.
- **Auditing complex, high-risk protocols** where there is little precedent or existing security literature.
- **Protecting millions in TVL** by ensuring robust security posture from foundational architecture to ongoing advisory.

---

## Notable Security Review Outcomes

### 1. DeFi & Smart Contracts
- **RadicalxChange** - **[3rd Rank](https://github.com/sherlock-audit/2024-02-radicalxchange-judging/issues/93)** <img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/b5634942-f69e-4f15-a23d-8297b7009e0f" style="vertical-align: middle;" />: Missing validation check allowed the highest bidder to prematurely withdraw collateral. 🏆
- **Sorella Angstrom** - **[4th Rank](https://cantina.xyz/code/84df57a3-0526-49b8-a7c5-334888f43940/overview/leaderboard)** <img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/b5634942-f69e-4f15-a23d-8297b7009e0f" style="vertical-align: middle;" /> (Patrol team): Arbitrary fee extraction via manipulation of initialized ticks.
- **Geneius Contracts** - **[6th Rank](https://cantina.xyz/code/12acc80c-4e4c-4081-a0a3-faa92150651a/overview/leaderboard)** <img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/b5634942-f69e-4f15-a23d-8297b7009e0f" style="vertical-align: middle;" />: Solana DoS vulnerability caused by `order_hash` collisions during order fulfillment.

### 2. Infrastructure & ZK Systems
- **[redacted]** (zkVM) - **1 Critical**: Identified soundness vulnerability in core proof verification logic.
- **GOATNetwork** - **[1 High](/blog/goat-slash-fraction-bypass/)**: Logic flaw in slash fraction calculations allowing negative slashing, validator inflation, and token siphoning.
- **LeanMultisig / LeanEthereum** (zkVM) - **[1 High, 2 Medium](/blog/lean-multisig-bugs/)**: Uncovered transcript collisions in the Fiat-Shamir backend.
- **[redacted]** (FHE) - **1 High & 2 Medium**: Discovered predictable on-chain randomness and associated logic flaws.

*See the [Audits](/audits/) tab for a full list of security reviews.*

---

## What I Work On

### 1. Smart Contracts & Execution Environments
Auditing complex AMMs, lending protocols, vaults, and liquid staking systems. Deep expertise in finding logic flaws across diverse execution environments, including **Solidity (EVM), Rust (Solana), Go (Cosmos), and Bitcoin Rollups**.

### 2. Infrastructure & Consensus
Deep design-level review of network architecture, cross-chain messaging, and sequencer security. Securing **L1/L2 systems, Custom VMs, and Bridges**, focusing on both underlying implementation and cryptographic integrity.

### 3. ZK Systems & Applied Cryptography
Uncovering soundness bugs, transcript collisions, and polynomial commitment flaws. Extensive experience securing custom circuits (**Circom, Cairo, and Noir**), advanced **zkVM** backends, and **Fully Homomorphic Encryption (FHE)** protocols.

---

## Open Source & Research

To support the security ecosystem, I build extensive tooling for the layers I audit:

- **[ZP1](https://github.com/ZippelLabs/ZP1)**: Core contributor to the **ZippelLabs zkVM**, focusing on proof generation performance and verifier security.
- **[Circom](https://github.com/this-vishalsingh/Circom-Security) / [Cairo](https://github.com/this-vishalsingh/Cairo-Security) Security**: Comprehensive databases of vulnerability patterns for ZK circuits and STARK systems.
- **[zkVM-Security](https://github.com/this-vishalsingh/zkVM-Security) & [FHE-Security](https://github.com/this-vishalsingh/FHE-Security)**: Specialized vulnerability patterns and security-review checklists for frontier runtimes and encryption schemes.
- **[SuperAudit](https://github.com/SuperAudit/SuperAudit-Plugin)**: AI Security Agent designed for automated vulnerability discovery.

*Explore the [Research](/research/) tab for more tools and deep-dives.*

---

## Contact

Interested in securing your AMM, L2 Rollup, or ZK circuit? Reach out directly to book an audit or strategy call:

- Email: [thisvishalsingh@gmail.com](mailto:thisvishalsingh@gmail.com)
- Telegram: [@thisvishalsingh](https://t.me/thisvishalsingh)
- X: [@thisvishalsingh](https://x.com/thisvishalsingh)
- Book a call: [Schedule here](https://calendar.app.google/T47tbEv9ta2D1gws5)
