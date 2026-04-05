---
layout: post
title: "Nova and CycleFold"
description: "Nova turns “prove a long computation” into “prove one step and fold it into a running claim,” using a folding scheme for committed relaxed R1CS. CycleFold is a clean way to use a cycle of curves so that these elliptic-curve costs don’t bloat the recursive verifier circuit."
date: 2026-03-31
tags: [zk, research]
permalink: /blog/nova-cyclefold/
active: blog
---

## The problem Nova solves: IVC
To efficiently prove repeated application of a step function,y = F ^(k)^(x) = F(F(F(x))),
a proof must show:
- each step was computed correctly, and
- the prover’s state already accounts for the correctness of all prior steps.

That’s **Incrementally Verifiable Computation (IVC)**: the prover maintains an artifact that grows *logically* with the number of steps, while remaining compact and easy to verify.

Nova achieves IVC via a **non-interactive folding scheme** over **committed relaxed R1CS**. At each step, the prover proves (1) the current incremental computation and (2) a fold that incorporates the previous step’s R1CS instance into a running relaxed-R1CS accumulator. The resulting verifier circuit is **constant-sized**, and its dominant cost is **two group scalar multiplications**.

On the prover side, each step is dominated by **two multiexponentiations**. Importantly, Nova **does not require a trusted setup** and **does not use FFTs**, which makes it practical to instantiate over a wide range of elliptic-curve cycles (as long as discrete log is hard).

## Folding scheme for Committed Relaxed R1CS

### What “folding” means
A **folding scheme** is a protocol where a prover and verifier take two NP instance–witness pairs`(U1, W1)` and `(U2, W2)` of the **same shape** (i.e., same constraint system / same `A,B,C`), and produce one folded pair `(U, W)`.

Informally: the folded instance should be satisfiable only if the originals were satisfiable
(except with negligible probability over the random challenge used in the fold).

**Intuition:** folding compresses two claims into one claim while preserving soundness.

**Figure 1 — Folding as a 2→1 reduction (verifier side vs prover side).**

<div style="text-align: center; margin: 2rem 0;">
  <img src="https://hackmd.io/_uploads/rkiZKwgOZg.png" alt="r1csfolding" style="width: 100%; max-width: 600px; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.15);">
</div>

Nova builds folding for R1CS, but it first needs a variant of R1CS that folds cleanly.

---

### Why “relaxed R1CS” exists (why vanilla R1CS doesn’t fold)
Vanilla R1CS satisfiability is:

$$ (A z) \circ (B z) = (C z) $$

If we try to fold by setting `z := z1 + r*z2`, then we expand:

$$ A(z_1 + r z_2) \circ B(z_1 + r z_2) = (A z_1 \circ B z_1) + r(A z_1 \circ B z_2 + A z_2 \circ B z_1) + r^2(A z_2 \circ B z_2).
$$

The middle term is a **cross-term**, and it prevents linear folding from preserving satisfiability.

Nova fixes this by introducing **relaxed R1CS**, which adds:
- an error/slack vector `E` to absorb cross-terms, and
- a scalar `u` to handle constant-term drift under folding.

A relaxed R1CS instance is satisfiable if:

$$ (A Z) \circ (B Z) = u * (C Z) + E, \quad Z = (W, x, u). $$

Ordinary R1CS is the special case `u = 1` and `E = 0`.

---

### Folding relaxed R1CS (where the cross-term `T` comes from)
To fold two relaxed instances, the verifier samples a random challenge `r`, and both sides set:

- `u := u1 + r*u2`
- `x := x1 + r*x2`

The prover computes a cross-term vector `T` that captures the mixed products:

$$
T := (A Z_1 \circ B Z_2) + (A Z_2 \circ B Z_1) - u1*(C Z_2) - u2*(C Z_1).
$$

Then the folded error becomes:

$$ E := E_1 + r*T + r^2*E_2. $$

**Figure 2 — Interactive fold of relaxed R1CS (P computes `T`, V picks `r`).**
<div style="text-align: center; margin: 2rem 0;">
  <img src="https://hackmd.io/_uploads/S1r7KPldZg.png" alt="foldingRelaxedR1Cs" style="width: 100%; max-width: 600px; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.15);">
</div>


---

### Folding committed relaxed R1CS (succinct + hides witness pieces)
Folding shouldn’t require sending full witnesses, and we usually want (parts of) zero-knowledge. Nova therefore **commits** to the large witness pieces `W` and the slack vector `E` using a succinct, hiding, additively-homomorphic commitment, and treats the openings as the witness. So the **instance** carries commitments $\bar{W}, \bar{E}$, while the **witness** carries their openings.

The fold updates commitments homomorphically:
- $\bar{W} := \bar{W}+ r*\bar{W}$
- $\bar{E} := \bar{E} + r*\bar{T} + r^2*\bar{E}$, where $\bar{T} = Com(T)$

**Figure 3 — Folding committed relaxed R1CS (send $\bar{T}$, fold commitments).**
<div style="text-align: center; margin: 2rem 0;">
  <img src="https://hackmd.io/_uploads/BkO4tvxdZg.png" alt="FoldingCRR1Cs" style="width: 100%; max-width: 600px; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.15);">
</div>


---

### Making folding non-interactive (Fiat–Shamir)
The fold above is interactive because the verifier chooses `r`.
Nova applies the (strong) Fiat–Shamir transform to derive `r` from a hash of the transcript,
yielding a **non-interactive folding scheme (NIFS)** in the random-oracle model
(and in practice instantiating the RO with a hash function).

**Transition:** Now we can fold non-interactively. Next we wrap this inside an augmented step circuit `F'` so each iteration both computes $F(z_i)$ and folds the new step into the running accumulator.


## Folding to IVC (single curve)

Nova turns a NIFS (non-interactive folding scheme) into IVC by defining an **augmented step computation** `F'` that does two things:
- compute one step of the target transition: `z_{i+1} = F(z_i)`
- fold the “current-step claim” into the running accumulator using the folding verifier:
  `U_acc,i+1 = Fold_V(u_i, U_acc,i)`

**Figure 4 — The augmented step circuit `F'` (Nova IVC, single curve).**

<div style="text-align: center; margin: 2rem 0;">
  <img src="https://hackmd.io/_uploads/SyHdrV5vZe.png" alt="augmentedconstraintspic" style="width: 100%; max-width: 600px; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.15);">
</div>

This is the circuit that runs at *every* IVC step. It takes as input the step index and current state, and it also takes the per-step instance `u_i` plus the running accumulator instance `U_acc,i`. It outputs the next state and updated proof objects.

**Inputs (left):**
- `i, z0, zi`: the step counter, initial state, and current state
- `u_i`: the “this step” R1CS instance (meant to be *strict*)
- `U_acc,i`: the accumulator instance encoding “all prior steps were valid”

**What the circuit enforces (bottom box):**
-  **Bind this step to the running accumulator (hash link).**
   It checks that the public input of the step instance matches a hash of the current transcript:`u_i.x = H(i, z0, zi, U_acc,i)`. This prevents mixing a `u_i` from a different step or a different accumulator.
- **Ensure the step instance is strict.**  
   It enforces that `u_i` is an ordinary (non-relaxed) R1CS instance by requiring `E = 0` and `u = 1`. (So only the accumulator is “relaxed”; the fresh step is “honest”.)
- **Fold the step into the accumulator.**  
   It runs the folding verifier inside the circuit: `U_acc,i+1 = NIFS.V(U_acc,i, u_i)`.  
   Intuition: this updates the running claim so it now represents “all steps up to i are valid”.
- **Prepare the next step’s hash binding.**  
   It computes the hash that the *next* step will check: `u_{i+1}.x = H(i+1, z0, z_{i+1}, U_acc,i+1)`. 

**Top box (computation):**
- Independently of folding, it computes the next state: `z_{i+1} = F(z_i)`.

**Outputs (right):**
- `i+1, z0, z_{i+1}` (state evolution)
- `u_{i+1}` (the next step’s strict instance, with its hash-bound public input)
- `U_acc,i+1` (updated accumulator after folding)

**Key takeaway:** `F'` turns “prove all steps so far” into “prove one step + fold”, while hash links ensure steps cannot be reordered or spliced across different accumulators.

### Decider (final check / compression)
After many folds, the verifier does **not** re-check every step. Instead, it runs a **decider** that checks the final accumulated instance is valid (often by wrapping it in a succinct SNARK / final proof system).

**Figure 5 — Decider checks the final accumulator (and may compress to a short proof).**
<div style="text-align: center; margin: 2rem 0;">
  <img src="https://hackmd.io/_uploads/BJIBYwg_Ze.png" alt="part1_novafolding2" style="width: 100%; max-width: 600px; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.15);">
</div>

---

## Nova implementation over a cycle of curves (why it helps)

### Why non-native arithmetic appears
In recursion, the augmented circuit must implement parts of the folding verifier. Those include **elliptic-curve commitment relations**, which are naturally expressed over the curve’s **base field**.

But the recursive circuit is usually defined over the curve’s **scalar field**. So you end up trying to do base-field EC arithmetic inside a scalar-field circuit, which becomes **non-native** and expensive.

**Figure — Two fields per curve:**
- base field $F_q(E)$ = coordinates live here
- scalar field $F_r(E)$ = scalars for group operations
<div style="text-align: center; margin: 2rem 0;">
  <img src="https://hackmd.io/_uploads/HJgEV7KvWg.png" alt="r1cs_curvecycles" style="width: 100%; max-width: 600px; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.15);">
</div>

### Cycle-of-curves idea
Use two curves $E_1, E_2$ such that their fields “swap”:

- $F_r(E_1) = F_q(E_2)$
- $F_r(E_2) = F_q(E_1)$

Then you can represent the verifier-circuit arithmetic natively by alternating which curve hosts the recursive step:
- the circuit over $F_r(E_1)$ can do arithmetic over $F_q(E_2)$  natively,
- and vice-versa.

Example cycle often used in practice: BN254 / Grumpkin.

**Figure 6 — Alternating recursion on a 2-cycle (each curve checks the other natively).**
<div style="text-align: center; margin: 2rem 0;">
  <img src="https://hackmd.io/_uploads/SJoO-2CD-g.png" alt="Nova_curvecycles" style="width: 100%; max-width: 600px; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.15);">
</div>



## CycleFold-based Nova
CycleFold starts from the observation that folding-scheme based IVCs (Nova/HyperNova/etc.) are “basically fine on one curve” except for a few elliptic-curve operations in the verifier circuit—particularly scalar multiplication + point addition—which naturally live in the curve’s base field, not its scalar field. If you try to do these inside a circuit over the “wrong” field, you need non-native arithmetic, which can explode to “a million multiplication gates or more”.

Instead of Nova’s 2-cycle approach(which needs ~10,000 multiplication gates on both curves to encode verifier circuits),CycleFold
- builds tiny co-processor circuit C~EC~ over the scalar field of second curve E~2~, where (because of the cycle) this arithmetic becomes native for operations over 𝐸~1~’s base field.
- 𝐶~𝐸𝐶~ is small in practice: about 1,000–1,500 multiplication gates.

<div style="text-align: center; margin: 2rem 0;">
  <img src="https://hackmd.io/_uploads/r1cdJKFvWe.png" alt="cyclefold2" style="width: 100%; max-width: 600px; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.15);">
</div>

- sonobe implementation of cyclefold based Nova

<div style="text-align: center; margin: 2rem 0;">
  <img src="https://sonobe.pse.dev/imgs/cyclefold-nova-diagram.png" alt="sonobe implementation" style="width: 100%; max-width: 600px; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.15);">
</div>

### Want to talk to an expert?
**To get full security manual-review and formal verification, [DM-tg](https://t.me/thisvishalsingh) or [Request Quote via ZippelLabs](https://zippellabs.github.io/)**

### Disclaimer

This is provided for educational purposes.

## Thanks

Some resources and images are taken from others and all thanks to open-source resources, that helps us in doing this research!!
