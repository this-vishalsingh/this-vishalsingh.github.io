---
layout: default
title: Home
active: home
---

### Core Contributor at [ZippelLabs](https://zippellabs.github.io/) - [ZP1](https://github.com/ZippelLabs/ZP1) zkVM

---

## Profile

I’m a blockchain security engineer with 2+ years of experience.

**[ZippelLabs](https://zippellabs.github.io/)** is the name I use for my cryptography research work. I focus on Zero-Knowledge Proofs, zero-knowledge Virtual Machines, and privacy protocols. I’m drawn to problems that resist easy explanations and require sustained technical engagement.

I am most effective when working with systems that are new, unconventional, or pushing the boundaries of what has been done before:

- **Finding deep, design-level bugs** that aren’t caught by standard checklists or automated tools.
- **Auditing novel, high-risk protocols** where there is little precedent or existing research.
- **Helping teams mature their security practices**, from foundational architecture to ongoing advisory.

---

## Notable Security Review Outcomes

- **[redacted]** (zkVM) — **1 Critical** : Soundness impact identified in core proof verification logic. ⚡
- **[redacted]** (FHE) — **1 High** : Vulnerability in ciphertext rotation and noise management. 🔐
- **leanEthereum / leanMultisig (zkVM)** — **[1 High, 2 Medium](/blog/lean-multisig-bugs/)** : Deciphered critical **Transcript Collisions (H-4)** in Fiat-Shamir backend through Zero-Padded Scalar Absorption. ⚡
- **GOATNetwork** — **1 High** : Identified logic flaw in **Slash Fraction calculation** allowing negative slashing, validator inflation, and token siphoning. 🛡️
- **RadicalxChange** — **3rd Rank** <img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/b5634942-f69e-4f15-a23d-8297b7009e0f" style="vertical-align: middle;" /> : [Highest bidder can withdraw his collateral](https://github.com/sherlock-audit/2024-02-radicalxchange-judging/issues/93) due to a missing check. 🏆
- **Sorella Angstrom** — **4th Rank** (Patrol team) : [Fees can be stolen by changing the initialized ticks](https://cantina.xyz/code/84df57a3-0526-49b8-a7c5-334888f43940/overview/leaderboard) 🎯
- **Geneius Contracts** — **6th Rank** : [DoS in Solana via order_hash collision](https://cantina.xyz/code/12acc80c-4e4c-4081-a0a3-faa92150651a/overview/leaderboard) during filling orders. 🛠️
- **Nitro-Labs/Solaxy** — **8th Rank** : [Stale owner index lets attackers create “ghost” programs](https://cantina.xyz/code/50d38b86-80a0-49af-9df8-70d7d601b7d7/overview/leaderboard) and exhaust resources. 👻
- **Napier** — **9th Rank** : [Loss of funds due to not collecting fees](https://github.com/sherlock-audit/2024-06-new-scope-judging/issues/364) during integration. 💸
- **FarcasterAttestation** — **16th Rank** : [The lack of ERC-165 compliance](https://cantina.xyz/code/f9326d2b-bb99-45a9-88c5-94c54aa1823a/overview/leaderboard) leads to integration failures. 🔗

*See the [Audits](/audits/) tab for a full list of security reviews.*

---

## Open Source & Research

- **[ZP1](https://github.com/ZippelLabs/ZP1)** — Core contributor to the **ZippelLabs zkVM**, focusing on proof generation performance and verifier security. 🧱
- **[Circom](https://github.com/this-vishalsingh/Circom-Security) / [Cairo](https://github.com/this-vishalsingh/Cairo-Security) Security** — Building comprehensive databases of vulnerability patterns for ZK circuits and STARK systems. 🕵️‍♂️
- **[zkVM-Security](https://github.com/this-vishalsingh/zkVM-Security)** — Comprehensive security analysis and vulnerability patterns for Zero-Knowledge virtual machines. ⚙️
- **[FHE-Security](https://github.com/this-vishalsingh/FHE-Security)** — Research notes on Fully Homomorphic Encryption implementation pitfalls and security checklists. 🔐
- **[ZKP-Audits-0xide](https://github.com/this-vishalsingh/ZKP-Audits-0xide)** — A specialized framework for auditing Zero-Knowledge Proof systems and cryptographic primitives. 🗺️
- **[ZSentinel](https://github.com/this-vishalsingh/ZSentinel)** — Real-time security monitoring and alerting for persistent ZK proof systems. 📡
- **[SuperAudit](https://github.com/SuperAudit/SuperAudit-Plugin)** — Next-generation AI Security Agent designed for automated vulnerability discovery. 🤖

*Explore the [Research](/research/) and [Development](/development/) tabs for more tools.*

---

## What I Work On

- **Smart Contract Security** — Comprehensive reviews of **AMM, Lending protocols, Vaults**, and **Liquid Staking** systems to ensure multi-million dollar TVL remains secure. 🔐
- **Blockchain Infrastructure** — Auditing of **L1/L2 systems, Rollups, Bridges**, and **zkVMs**, focusing on both implementation and cryptographic integrity. 🏗️
- **Zero-Knowledge Proofs (ZKP)** — Expertise in securing **Circom, Cairo, and Noir** circuits, as well as core components like **Fiat-Shamir transcripts** and **Polynomial Commitments**. 🧙‍♂️
- **Multi-Chain Resilience** — Expertise in securing protocols across **Ethereum, Solana, and Cosmos**, adapting security mental models to diverse execution environments. 🌐

---

*Content last updated March 2026.*

## Contact

Interested in discussing ZK security, private audits, or research? 

- **Email:** [thisvishalsingh@gmail.com](mailto:thisvishalsingh@gmail.com)
- **Telegram:** [@thisvishalsingh](https://t.me/thisvishalsingh)
- **X/Twitter:** [@thisvishalsingh](https://x.com/thisvishalsingh)
- **Book a call:** [30 min call](https://calendar.app.google/T47tbEv9ta2D1gws5)
