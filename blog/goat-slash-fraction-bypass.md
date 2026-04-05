---
layout: post
title: "GOAT Network — Slash Fraction Validation Bypass"
description: "A one-character field swap in GOAT Network's slashing parameter validator silently disabled the negativity guard on the most severe penalty parameter. A governance proposal could set SlashFractionDoubleSign to a negative value, and every subsequent double-sign evidence event would *increase* the offending validator's stake rather than reduce it — printing BTC-backed collateral from nothing."
date: 2026-04-03
tags: [cosmos-sdk, bug-bounty, blockchain-security, token-inflation]
permalink: /blog/goat-slash-fraction-bypass/
active: blog
---

In this post

1. [Bitcoin's scaling problem and L2s](#l2)
2. [Cosmos SDK chains — how modules own state](#cosmos)
3. [GOAT Network architecture](#architecture)
4. [How slashing works end-to-end](#slashing)
5. [The bug — formal proof](#bug)
6. [Mathematical proof of token inflation](#math)
7. [Attack flow](#attack)
8. [Proof of concept](#poc)
9. [The fix](#fix)
10. [Summary](#summary)

---

## Bitcoin's scaling problem and L2s {#l2}

Bitcoin's base layer processes roughly 7 transactions per second. Every transaction
competes for space in a 1 MB block produced every ~10 minutes. This ceiling is
intentional: Bitcoin optimizes for finality and censorship resistance, not
throughput. The trade-off works for high-value settlement, but it rules Bitcoin out
as a platform for DeFi, smart contracts, or any application that needs sub-minute
confirmation.

Layer 2 networks address this by separating execution from settlement. Transactions
run on a secondary chain — faster and cheaper — while Bitcoin's base layer remains
the trust anchor. The hard problem for any Bitcoin L2 is *how* it inherits
Bitcoin's security. Projects answer this differently:

- **State channels** (Lightning) — bilateral off-chain accounting, settled on
  dispute.
- **Merged-mined sidechains** — miner participation as the security bridge.
- **ZK rollups** — batch many L2 transactions into a single ZK proof that
  Bitcoin can verify.

GOAT Network takes the ZK rollup approach. All transactions execute on the L2
network. Their validity is proven with zero-knowledge proofs. Bitcoin miners are
the final arbiters: if a sequencer posts an invalid state root, anyone can submit a
fraud proof and Bitcoin's scripting layer enforces the correct outcome via BitVM2.

```
┌───────────────── Bitcoin L1 (settlement + dispute) ──────────────────┐
│  BitVM2 bridge         ZK proof anchor         Miner enforcement      │
└──────────────────────────────┬───────────────────────────────────────┘
                               │ ZK proofs committed
┌──────────────────────────────▼───────────────────────────────────────┐
│              ZKM zkVM prover network (GPU, Groth16)                  │
└──────────────────────────────┬───────────────────────────────────────┘
                               │ batch validity
┌──────────────────────────────▼───────────────────────────────────────┐
│           GOAT L2 — Cosmos SDK chain (CometBFT consensus)            │
│  x/staking · x/evidence · x/slashing · x/gov · x/locking ★          │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Cosmos SDK chains — how modules own state {#cosmos}

The Cosmos SDK is a Go framework for building application-specific blockchains.
Rather than deploying a smart contract on a shared chain, teams ship their own
sovereign chain — with validators, consensus, tokenomics, and governance —
assembled from composable modules. Each module owns a slice of the chain's KV
store and exposes message handlers, query endpoints, and genesis configuration.

```
CometBFT consensus engine
         │
         ▼
   ABCI application
   ├── x/staking    — validator set, delegation, bonded tokens
   ├── x/evidence   — routes misbehavior proofs to handlers
   ├── x/slashing   — signing records, executes stake reductions
   ├── x/gov        — on-chain proposals, the only path to change Params
   └── x/locking ★  — GOAT custom: BTC collateral, SlashFraction params  ← BUG
```

**This matters for severity.** Unlike a smart contract, Cosmos SDK module logic
runs inside the chain's state machine itself. A bug in module code runs as
consensus-level code on every full node simultaneously. There is no EVM sandbox.
Every node executes the same state transition, including the wrong one.

CometBFT achieves Byzantine fault-tolerant finality in ~1–6 seconds. Validators
participate by bonding stake. If a validator misbehaves, the slashing module burns
a fraction of their bonded tokens — deterministically enforced by every node
running the same `Params`.

---

## GOAT Network architecture {#architecture}

GOAT Network is structured as a ZK rollup whose execution environment and
validator coordination live on a Cosmos SDK chain, while dispute resolution and
final settlement are anchored to Bitcoin via BitVM2.

<svg width="100%" viewBox="0 0 680 580" xmlns="http://www.w3.org/2000/svg">
<defs>
<marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
<path d="M2 1L8 5L2 9" fill="none" stroke="#888780" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
</marker>
<marker id="arr-r" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
<path d="M2 1L8 5L2 9" fill="none" stroke="#D85A30" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
</marker>
</defs>
<rect x="30" y="20" width="620" height="110" rx="12" fill="#F1EFE8" stroke="#888780" stroke-width="0.5"/>
<text x="340" y="46" text-anchor="middle" font-family="monospace" font-size="13" font-weight="500" fill="#444441">Bitcoin Layer 1 — settlement and dispute enforcement</text>
<text x="340" y="64" text-anchor="middle" font-family="monospace" font-size="11" fill="#5F5E5A">Miners validate disputes · ZK proofs anchored via BitVM2 · Permissionless BTC exits</text>
<rect x="110" y="78" width="140" height="36" rx="6" fill="#E6F1FB" stroke="#378ADD" stroke-width="0.5"/>
<text x="180" y="96" text-anchor="middle" font-family="monospace" font-size="12" font-weight="500" fill="#0C447C">BitVM2 bridge</text>
<text x="180" y="109" text-anchor="middle" font-family="monospace" font-size="10" fill="#185FA5">Trustless BTC exits</text>
<rect x="430" y="78" width="140" height="36" rx="6" fill="#E6F1FB" stroke="#378ADD" stroke-width="0.5"/>
<text x="500" y="96" text-anchor="middle" font-family="monospace" font-size="12" font-weight="500" fill="#0C447C">ZK proof anchor</text>
<text x="500" y="109" text-anchor="middle" font-family="monospace" font-size="10" fill="#185FA5">Batch validity on L1</text>
<line x1="340" y1="130" x2="340" y2="155" stroke="#888780" stroke-width="1" marker-end="url(#arr)"/>
<rect x="30" y="157" width="620" height="42" rx="10" fill="#E1F5EE" stroke="#1D9E75" stroke-width="0.5"/>
<text x="340" y="176" text-anchor="middle" font-family="monospace" font-size="12" font-weight="500" fill="#085041">ZKM zkVM prover network</text>
<text x="340" y="191" text-anchor="middle" font-family="monospace" font-size="10" fill="#0F6E56">Distributed GPU nodes · Groth16 proofs in parallel · sub-3s proof generation</text>
<line x1="340" y1="199" x2="340" y2="224" stroke="#888780" stroke-width="1" marker-end="url(#arr)"/>
<rect x="20" y="226" width="640" height="330" rx="14" fill="#EEEDFE" stroke="#534AB7" stroke-width="0.5"/>
<text x="340" y="250" text-anchor="middle" font-family="monospace" font-size="13" font-weight="500" fill="#3C3489">GOAT L2 — Cosmos SDK chain (CometBFT BFT consensus, EVM-compatible)</text>
<text x="340" y="267" text-anchor="middle" font-family="monospace" font-size="10" fill="#534AB7">Validator set managed by bonded BTC collateral (yBTC). Governance controls Params.</text>
<rect x="44" y="280" width="128" height="48" rx="6" fill="#F1EFE8" stroke="#888780" stroke-width="0.5"/>
<text x="108" y="300" text-anchor="middle" font-family="monospace" font-size="12" font-weight="500" fill="#2C2C2A">x/staking</text>
<text x="108" y="316" text-anchor="middle" font-family="monospace" font-size="10" fill="#5F5E5A">validators, delegation</text>
<rect x="186" y="280" width="128" height="48" rx="6" fill="#F1EFE8" stroke="#888780" stroke-width="0.5"/>
<text x="250" y="300" text-anchor="middle" font-family="monospace" font-size="12" font-weight="500" fill="#2C2C2A">x/evidence</text>
<text x="250" y="316" text-anchor="middle" font-family="monospace" font-size="10" fill="#5F5E5A">routes misbehavior</text>
<rect x="328" y="280" width="128" height="48" rx="6" fill="#F1EFE8" stroke="#888780" stroke-width="0.5"/>
<text x="392" y="300" text-anchor="middle" font-family="monospace" font-size="12" font-weight="500" fill="#2C2C2A">x/slashing</text>
<text x="392" y="316" text-anchor="middle" font-family="monospace" font-size="10" fill="#5F5E5A">signing records</text>
<rect x="470" y="280" width="128" height="48" rx="6" fill="#F1EFE8" stroke="#888780" stroke-width="0.5"/>
<text x="534" y="300" text-anchor="middle" font-family="monospace" font-size="12" font-weight="500" fill="#2C2C2A">x/gov</text>
<text x="534" y="316" text-anchor="middle" font-family="monospace" font-size="10" fill="#5F5E5A">param proposals</text>
<rect x="160" y="350" width="360" height="60" rx="10" fill="#FAECE7" stroke="#D85A30" stroke-width="1.2"/>
<text x="340" y="372" text-anchor="middle" font-family="monospace" font-size="13" font-weight="500" fill="#712B13">x/locking ★ — custom BTC collateral module</text>
<text x="340" y="388" text-anchor="middle" font-family="monospace" font-size="10" fill="#993C1D">Holds yBTC · sequencer stake · SlashFraction params · Params.Validate() ← BUG</text>
<text x="340" y="402" text-anchor="middle" font-family="monospace" font-size="10" fill="#D85A30">SlashFractionDoubleSign not guarded against negative values</text>
<line x1="534" y1="328" x2="450" y2="348" stroke="#D85A30" stroke-width="1" stroke-dasharray="4 3" marker-end="url(#arr-r)"/>
<line x1="250" y1="328" x2="280" y2="348" stroke="#888780" stroke-width="0.8" stroke-dasharray="4 3" marker-end="url(#arr)"/>
<line x1="392" y1="328" x2="370" y2="348" stroke="#888780" stroke-width="0.8" stroke-dasharray="4 3" marker-end="url(#arr)"/>
<rect x="60" y="430" width="560" height="90" rx="8" fill="#F1EFE8" stroke="#888780" stroke-width="0.5"/>
<text x="340" y="450" text-anchor="middle" font-family="monospace" font-size="12" font-weight="500" fill="#2C2C2A">Sequencer validator set</text>
<text x="340" y="468" text-anchor="middle" font-family="monospace" font-size="10" fill="#5F5E5A">BTC locked as yBTC collateral · ranked by stake · propose + sign L2 blocks</text>
<text x="340" y="485" text-anchor="middle" font-family="monospace" font-size="10" fill="#5F5E5A">double-sign evidence → handleEvidence → slashAmount = Tokens × SlashFractionDoubleSign</text>
<text x="340" y="501" text-anchor="middle" font-family="monospace" font-size="10" fill="#D85A30">if SlashFractionDoubleSign &lt; 0 → slashAmount &lt; 0 → Tokens increase (token inflation)</text>
<line x1="340" y1="410" x2="340" y2="428" stroke="#888780" stroke-width="1" marker-end="url(#arr)"/>
<text x="36" y="571" font-family="monospace" font-size="10" fill="#888780">★ x/locking is GOAT-custom · all other modules are standard Cosmos SDK</text>
</svg>

**Three distinct layers, one trust chain.** Bitcoin provides the trust root. The
ZKM proof layer provides cryptographic validity. The Cosmos SDK chain provides the
execution environment and economic coordination. The vulnerability lives in the
Cosmos SDK layer — specifically in how governance parameters are validated before
they reach the execution environment.

### The sequencer and the locking module

On GOAT Network, sequencers must lock BTC as collateral via `x/locking`. This
collateral secures two things: it makes misbehavior costly, and it provides
native BTC yield to operators through gas fees and GOAT token rewards. When a
sequencer locks BTC, a `yBTC` token is minted on L2 as a receipt. The validator
set is ranked by locked collateral. Governance — via `x/gov` — controls the
slashing parameters.

Two critical parameters live in `x/locking/types/params.go`:

```go
type Params struct {
    SlashFractionDoubleSign sdk.Dec  // fraction burned on equivocation (double-sign)
    SlashFractionDowntime   sdk.Dec  // fraction burned on liveness failure
    // ...
}
```

Both are `sdk.Dec` — arbitrary-precision signed decimal. Typical values:
`SlashFractionDoubleSign = 0.05` (5%), `SlashFractionDowntime = 0.0001` (0.01%).

---

## How slashing works end-to-end {#slashing}

When a validator double-signs — producing two conflicting vote messages for the
same block height — CometBFT detects the conflict and the evidence is processed
through the following path:

```
1. CometBFT gossip detects conflicting pre-commits at height H
         │
         ▼
2. DuplicateVoteEvidence packaged → MsgSubmitEvidence submitted
         │
         ▼
3. x/evidence module validates format, routes to handleEvidence
         │
         ▼
4. Keeper reads SlashFractionDoubleSign from Params (KV store)
         │
         ▼
5. slashAmount = validator.Tokens × SlashFractionDoubleSign
         │
         ▼
6. validator.Tokens -= slashAmount   (expected: tokens decrease)
   totalSlashed     += slashAmount
         │
         ▼
7. Validator tombstoned (permanent jail for double-sign)
```

The keeper **trusts** that `Params.Validate()` enforced the invariant
`SlashFractionDoubleSign ∈ (0, 1]` before the value reached the KV store. This
trust is the assumption the bug breaks.

---

## The bug — formal proof {#bug}

### Invariant definition

Let $\mathcal{P}$ be the set of valid `Params` values. The slashing module's
safety invariant requires:

$$\mathcal{I}_{\text{slash}} : \forall p \in \mathcal{P},\quad 0 < p.\texttt{SlashFractionDoubleSign} \leq 1 \;\land\; 0 < p.\texttt{SlashFractionDowntime} \leq 1$$

`Params.Validate()` is the sole enforcement gate for this invariant. It is called
at genesis and on every governance parameter-change proposal.

### The defective guard

The actual code before the patch:

```go
func (p Params) Validate() error {
    // ...
    if p.SlashFractionDoubleSign.IsZero() || p.SlashFractionDowntime.IsNegative() {
        return fmt.Errorf("SlashFractionDoubleSign too low: %s",
            p.SlashFractionDoubleSign.String())
    }
    // ...
}
```

The guard constructs a compound condition:

$$G(p) \equiv \bigl(p.\texttt{SFDS} = 0\bigr) \;\lor\; \bigl(p.\texttt{SFDt} < 0\bigr)$$

where $\texttt{SFDS} = \texttt{SlashFractionDoubleSign}$ and $\texttt{SFDt} = \texttt{SlashFractionDowntime}$.

### Proof of guard failure

**Claim:** There exists a value of $p.\texttt{SFDS}$ that violates
$\mathcal{I}_{\text{slash}}$ yet satisfies $\neg G(p)$, causing `Validate()` to
return `nil`.

**Proof.** Choose any $p$ such that:

$$p.\texttt{SFDS} = -\delta \quad \text{for some } \delta > 0, \qquad p.\texttt{SFDt} = \varepsilon \quad \text{for some } \varepsilon > 0$$

Evaluate $G(p)$:

$$\bigl(p.\texttt{SFDS} = 0\bigr) = \bigl(-\delta = 0\bigr) = \textbf{false}$$
$$\bigl(p.\texttt{SFDt} < 0\bigr) = \bigl(\varepsilon < 0\bigr) = \textbf{false}$$

$$\therefore\quad G(p) = \textbf{false} \lor \textbf{false} = \textbf{false}$$

The condition is `false`, so `Validate()` does **not** return an error. Yet:

$$p.\texttt{SFDS} = -\delta < 0 \implies p.\texttt{SFDS} \notin (0,\,1]$$

This violates $\mathcal{I}_{\text{slash}}$. $\blacksquare$

The correct guard should be:

$$G'(p) \equiv \bigl(p.\texttt{SFDS} \leq 0\bigr) \;\lor\; \bigl(p.\texttt{SFDS} > 1\bigr)$$

evaluated with the right variable name on every sub-expression.

---

## Mathematical proof of token inflation {#math}

### Setup

Let $T \in \mathbb{Z}_{>0}$ be the validator's bonded token balance. Let
$f = p.\texttt{SlashFractionDoubleSign} \in \mathbb{Q}$. The keeper computes:

$$\texttt{slashAmount} = \left\lfloor T \cdot f \right\rfloor$$

$$T' = T - \texttt{slashAmount} = T - \left\lfloor T \cdot f \right\rfloor$$

### Normal operation (f ∈ (0, 1])

For $f \in (0, 1]$:

$$T \cdot f > 0 \implies \left\lfloor T \cdot f \right\rfloor \geq 1$$

$$\therefore\quad T' = T - \left\lfloor T \cdot f \right\rfloor \leq T - 1 < T$$

The validator's balance strictly decreases. $\checkmark$

### Exploit case (f < 0)

For $f = -\delta$ where $\delta > 0$:

$$\texttt{slashAmount} = \left\lfloor T \cdot (-\delta) \right\rfloor = -\left\lceil T \cdot \delta \right\rceil$$

Since $T > 0$ and $\delta > 0$, we have $T \cdot \delta > 0$, so
$\left\lceil T \cdot \delta \right\rceil \geq 1$. Therefore:

$$\texttt{slashAmount} \leq -1 < 0$$

Substituting into the token update:

$$T' = T - \texttt{slashAmount} = T - \bigl(-\left\lceil T\delta \right\rceil\bigr) = T + \left\lceil T\delta \right\rceil$$

$$\boxed{T' = T + \left\lceil T \cdot \delta \right\rceil > T}$$

The validator's balance **strictly increases**. Each double-sign event mints
$\left\lceil T \cdot \delta \right\rceil$ tokens from nothing.

### Quantified impact

For a validator holding $T = 1{,}000{,}000$ tokens with $\delta = 0.05$:

$$\Delta T = \left\lceil 1{,}000{,}000 \times 0.05 \right\rceil = 50{,}000 \text{ tokens per slash event}$$

After $n$ iterations (unjail, re-stake, double-sign again):

$$T_n = T_0 + n \cdot \left\lceil T_0 \cdot \delta \right\rceil \approx T_0 \cdot (1 + n\delta)$$

The balance grows **linearly in $n$** with slope $T_0 \cdot \delta$. The attacker
can extract real BTC by redeeming inflated `yBTC` through the BitVM2 bridge until
the reserve is drained or the discrepancy is detected on-chain.

### Accounting invariant violation

The locking module maintains a total-slashed counter $S$. After $n$ events:

$$S_n = S_0 + n \cdot \texttt{slashAmount} = S_0 + n \cdot \bigl(-\left\lceil T\delta \right\rceil\bigr) < S_0$$

The counter *decreases*, which is economically nonsensical — negative total slashing
masks the inflation. On-chain accounting appears normal.

The broken invariant is:

$$\mathcal{I}_{\text{account}} : S_n \geq S_0 \;\text{ for all } n > 0$$

With $f < 0$ this invariant is violated on every single evidence event.

---

## Attack flow {#attack}

<svg width="100%" viewBox="0 0 680 620" xmlns="http://www.w3.org/2000/svg">
<defs>
<marker id="a2" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
<path d="M2 1L8 5L2 9" fill="none" stroke="#D85A30" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
</marker>
<marker id="a3" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
<path d="M2 1L8 5L2 9" fill="none" stroke="#888780" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
</marker>
</defs>
<text x="340" y="20" text-anchor="middle" font-family="monospace" font-size="12" fill="#888780">Five-step exploit chain</text>
<rect x="220" y="34" width="240" height="58" rx="8" fill="#FCEBEB" stroke="#E24B4A" stroke-width="1"/>
<text x="340" y="56" text-anchor="middle" font-family="monospace" font-size="12" font-weight="500" fill="#791F1F">Step 1 — acquire governance power</text>
<text x="340" y="72" text-anchor="middle" font-family="monospace" font-size="10" fill="#A32D2D">Accumulate / borrow delegation ≥ quorum</text>
<text x="340" y="85" text-anchor="middle" font-family="monospace" font-size="10" fill="#A32D2D">or collude with existing validators</text>
<line x1="340" y1="92" x2="340" y2="116" stroke="#D85A30" stroke-width="1.5" marker-end="url(#a2)"/>
<rect x="140" y="118" width="400" height="58" rx="8" fill="#FCEBEB" stroke="#E24B4A" stroke-width="1"/>
<text x="340" y="140" text-anchor="middle" font-family="monospace" font-size="12" font-weight="500" fill="#791F1F">Step 2 — MsgSubmitProposal with negative fraction</text>
<text x="340" y="156" text-anchor="middle" font-family="monospace" font-size="10" fill="#A32D2D">SlashFractionDoubleSign = sdk.NewDecWithPrec(-5, 2)  // -0.05</text>
<text x="340" y="169" text-anchor="middle" font-family="monospace" font-size="10" fill="#A32D2D">Validate() reads SlashFractionDowntime.IsNegative() → false → nil</text>
<line x1="340" y1="176" x2="340" y2="200" stroke="#D85A30" stroke-width="1.5" marker-end="url(#a2)"/>
<rect x="200" y="202" width="280" height="48" rx="8" fill="#FCEBEB" stroke="#E24B4A" stroke-width="1"/>
<text x="340" y="222" text-anchor="middle" font-family="monospace" font-size="12" font-weight="500" fill="#791F1F">Step 3 — negative param stored in KV</text>
<text x="340" y="237" text-anchor="middle" font-family="monospace" font-size="10" fill="#A32D2D">x/locking KV store now holds f = -0.05</text>
<line x1="340" y1="250" x2="340" y2="274" stroke="#D85A30" stroke-width="1.5" marker-end="url(#a2)"/>
<rect x="100" y="276" width="480" height="68" rx="8" fill="#FCEBEB" stroke="#E24B4A" stroke-width="1"/>
<text x="340" y="298" text-anchor="middle" font-family="monospace" font-size="12" font-weight="500" fill="#791F1F">Step 4 — trigger double-sign evidence</text>
<text x="340" y="314" text-anchor="middle" font-family="monospace" font-size="10" fill="#A32D2D">Attacker's validator signs two conflicting blocks at height H</text>
<text x="340" y="328" text-anchor="middle" font-family="monospace" font-size="10" fill="#A32D2D">handleEvidence: slashAmount = T × (-0.05) = -50,000   →   T' = T + 50,000</text>
<text x="340" y="341" text-anchor="middle" font-family="monospace" font-size="10" fill="#A32D2D">totalSlashed -= 50,000  (counter decreases — no alarm raised)</text>
<line x1="340" y1="344" x2="340" y2="368" stroke="#D85A30" stroke-width="1.5" marker-end="url(#a2)"/>
<rect x="140" y="370" width="400" height="58" rx="8" fill="#FCEBEB" stroke="#E24B4A" stroke-width="1"/>
<text x="340" y="392" text-anchor="middle" font-family="monospace" font-size="12" font-weight="500" fill="#791F1F">Step 5 — extract via BitVM2 bridge</text>
<text x="340" y="408" text-anchor="middle" font-family="monospace" font-size="10" fill="#A32D2D">Unbond inflated stake → redeem yBTC → BitVM2 withdrawal to BTC L1</text>
<text x="340" y="421" text-anchor="middle" font-family="monospace" font-size="10" fill="#A32D2D">Unjail under new key → repeat from Step 4</text>
<line x1="534" y1="399" x2="580" y2="399" stroke="#D85A30" stroke-width="1" stroke-dasharray="3 3"/>
<line x1="580" y1="399" x2="580" y2="180" stroke="#D85A30" stroke-width="1" stroke-dasharray="3 3"/>
<line x1="580" y1="180" x2="534" y2="180" stroke="#D85A30" stroke-width="1" stroke-dasharray="3 3" marker-end="url(#a2)"/>
<text x="598" y="296" text-anchor="middle" font-family="monospace" font-size="10" fill="#D85A30" transform="rotate(90,598,296)">repeat</text>
<rect x="30" y="458" width="620" height="64" rx="10" fill="#F1EFE8" stroke="#888780" stroke-width="0.5"/>
<text x="340" y="478" text-anchor="middle" font-family="monospace" font-size="12" font-weight="500" fill="#2C2C2A">Preconditions</text>
<text x="340" y="494" text-anchor="middle" font-family="monospace" font-size="10" fill="#5F5E5A">(1) Governance quorum or validator collusion</text>
<text x="340" y="508" text-anchor="middle" font-family="monospace" font-size="10" fill="#5F5E5A">(2) Control of a validator key (own node or key compromise)</text>
<rect x="30" y="540" width="620" height="64" rx="10" fill="#EAF3DE" stroke="#639922" stroke-width="0.5"/>
<text x="340" y="560" text-anchor="middle" font-family="monospace" font-size="12" font-weight="500" fill="#173404">Maximum at-risk surface</text>
<text x="340" y="576" text-anchor="middle" font-family="monospace" font-size="10" fill="#27500A">5,000 BTC committed collateral across GOAT launch operators</text>
<text x="340" y="590" text-anchor="middle" font-family="monospace" font-size="10" fill="#27500A">+ downstream DeFi protocols consuming yBTC as BTC proxy</text>
</svg>

---

## Proof of concept {#poc}

Run with:

```bash
go test ./x/locking/types/... -run TestNegativeDoubleSignBypass -v
```

```go
func TestNegativeDoubleSignBypass(t *testing.T) {
    // Construct params with a negative SlashFractionDoubleSign.
    // This SHOULD be rejected by Validate(). It is not — that is the bug.
    params := types.NewParams(
        sdk.NewDecWithPrec(-5, 2),   // SlashFractionDoubleSign = -0.05
        sdk.NewDecWithPrec(1, 1000), // SlashFractionDowntime   = normal
        // remaining params: genesis defaults
    )

    err := params.Validate()

    // Bug: err == nil. The negative value passes validation silently.
    require.Error(t, err,
        "expected Validate() to reject negative SlashFractionDoubleSign, got nil")
}
```

On the **unpatched** binary: the test fails (`require.Error` fires because `err == nil`).  
On the **patched** binary: `Validate()` returns
`"SlashFractionDoubleSign too low: -0.050000000000000000"` and the test passes.

To confirm the arithmetic inversion in the keeper:

```go
func TestInvertedSlash(t *testing.T) {
    initialTokens := sdk.NewInt(1_000_000)
    slashFactor   := sdk.NewDecWithPrec(-5, 2)   // -0.05

    slashAmount := initialTokens.ToDec().Mul(slashFactor).TruncateInt()
    newTokens   := initialTokens.Sub(slashAmount)

    // slashAmount  = -50,000
    // newTokens    = 1,050,000  ← tokens increased
    require.True(t, newTokens.GT(initialTokens),
        "expected token inflation: got %s from %s", newTokens, initialTokens)
}
```

---

## The fix {#fix}

Separate the two fraction guards so each references only its own field:

```diff
 func (p Params) Validate() error {
     // ...
-    if p.SlashFractionDoubleSign.IsZero() || p.SlashFractionDowntime.IsNegative() {
-        return fmt.Errorf("SlashFractionDoubleSign too low: %s",
-            p.SlashFractionDoubleSign.String())
-    }
+    if p.SlashFractionDoubleSign.IsZero() || p.SlashFractionDoubleSign.IsNegative() {
+        return fmt.Errorf("SlashFractionDoubleSign too low: %s",
+            p.SlashFractionDoubleSign.String())
+    }
+    if p.SlashFractionDowntime.IsZero() || p.SlashFractionDowntime.IsNegative() {
+        return fmt.Errorf("SlashFractionDowntime too low: %s",
+            p.SlashFractionDowntime.String())
+    }
     // ...
 }
```

### Why this is the correct fix

The patched guard enforces:

$$G'_{\text{DS}}(p) \equiv \bigl(p.\texttt{SFDS} = 0\bigr) \lor \bigl(p.\texttt{SFDS} < 0\bigr) \equiv p.\texttt{SFDS} \leq 0$$

This correctly rejects any $p.\texttt{SFDS} \leq 0$. Combined with an upper-bound
check (recommended below), the full invariant becomes enforceable:

$$G'_{\text{full}}(p) \equiv (p.\texttt{SFDS} \leq 0) \lor (p.\texttt{SFDS} > 1)$$

$$\neg G'_{\text{full}}(p) \iff p.\texttt{SFDS} \in (0,\,1] \qquad \checkmark$$

### Recommended hardening

Beyond the minimal fix, add upper-bound checks so neither fraction can exceed 1:

```go
if p.SlashFractionDoubleSign.GT(sdk.OneDec()) {
    return fmt.Errorf("SlashFractionDoubleSign too high: %s",
        p.SlashFractionDoubleSign.String())
}
if p.SlashFractionDowntime.GT(sdk.OneDec()) {
    return fmt.Errorf("SlashFractionDowntime too high: %s",
        p.SlashFractionDowntime.String())
}
```

Add defense-in-depth inside the keeper before applying the multiplication:

```go
// In handleEvidence, before slashing:
if slashFactor.IsNegative() || slashFactor.IsZero() || slashFactor.GT(sdk.OneDec()) {
    panic(fmt.Sprintf("invariant violation: invalid slash factor %s reached keeper", slashFactor))
}
```

And add table-driven tests that assert `Validate()` rejects each field independently:

```go
var validateTests = []struct {
    name   string
    params types.Params
    wantOK bool
}{
    {"double-sign negative",  paramsWithDSF(sdk.NewDecWithPrec(-1, 2)),  false},
    {"double-sign zero",      paramsWithDSF(sdk.ZeroDec()),              false},
    {"double-sign above one", paramsWithDSF(sdk.NewDecWithPrec(11, 1)), false},
    {"downtime negative",     paramsWithDT(sdk.NewDecWithPrec(-1, 4)),   false},
    {"both valid",            validParams(),                              true},
}
```

---

## Summary {#summary}

| Field | Finding | Severity | Status |
|---|---|---|---|
| `SlashFractionDoubleSign` | Validate() checks wrong field for IsNegative() | High | Fixed |
| `SlashFractionDowntime` | Upper-bound check absent (hardening) | Low | Recommended |

**Root cause:** A copy-paste error in a compound validation condition. Two out of
three sub-expressions named the correct variable; the one responsible for the
negative check named the wrong one. Long, similar variable names make this
invisible to casual code review.

**Lesson for auditors:** Governance-controlled parameters are a live attack surface.
Any `sdk.Dec` that flows from a proposal into arithmetic — without re-validation at
the execution site — is a potential invariant violation. When reviewing `Validate()`
functions, verify each sub-expression references the variable named in the error
message. A mismatch between the error string and the guarded variable is a
near-certain indicator of this class of bug.

---

Patch: [GOATNetwork/goat@091da717](https://github.com/GOATNetwork/goat/commit/091da717ef99f99fa5066cc90e1e3908dd0477c0)  
Author: [@this-vishalsingh](https://github.com/this-vishalsingh)
