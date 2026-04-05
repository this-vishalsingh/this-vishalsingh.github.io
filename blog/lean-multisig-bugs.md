---
layout: post
title: "Breaking LeanMultisig zkVM Shamir (Part 1)"
description: "A formally-verified circuit is only as secure as the transcript feeding it randomness. I found three bugs in LeanMultisig's Fiat-Shamir layer that prove this — one that lets a prover recycle an old proof against a different public input, two that silently corrupt or bypass security checks on 32-bit and max-difficulty targets. All three share the same root cause: trusting the caller to supply well-formed inputs."
date: 2026-03-28
tags: ["zk", "vuln"]
permalink: /blog/lean-multisig-bugs/
active: blog
---

<!-- TOC -->
<div class="toc-wrap">
  <div class="toc">
    <div class="toc-head">In this post</div>
    <ol>
      <li><a href="#story">How I got here</a></li>
      <li><a href="#what-is-fs">What Fiat-Shamir actually is — with the math</a></li>
      <li><a href="#sponge">How the sponge absorbs scalars</a></li>
      <li><a href="#h4">H-4 — The collision bug that broke statement binding</a></li>
      <li><a href="#m1">M-1 — The bitmask that panics on 32-bit</a></li>
      <li><a href="#m2">M-2 — The overflow that turns PoW into a no-op</a></li>
      <li><a href="#lessons">What I learned — and what you should check in your own code</a></li>
      <li><a href="#summary">Summary</a></li>
    </ol>
  </div>
</div>

<!-- STORY -->
<h2 id="story">How I got here</h2>

<p>
  I had been reading the leanEthereum codebase for a few days — not looking for anything in particular, just mapping how the proving pipeline works end-to-end. The high-level circuits are beautiful. The Lean 4 formalization is genuinely impressive. But the more I read, the more I kept coming back to one folder: <code>crates/backend/fiat-shamir/</code>.
</p>

<div class="personal">
  Every time I audit a ZK system, I start with the transcript. Not the circuits. Not the constraint system. The transcript. A circuit can be perfectly specified and formally verified, and the whole system still falls apart if the randomness feeding it is not unique. The transcript is the one place where a subtle mistake becomes a catastrophic one.
</div>

<p>
  A day reading <code>challenger.rs</code> and <code>verifier.rs</code> turned up three bugs — different in severity, all rooted in the same assumption: that callers supply well-formed inputs. In a proof system, that assumption should never hold implicitly.
</p>

<!-- FIAT-SHAMIR PRIMER -->
<h2 id="what-is-fs">What Fiat-Shamir actually is — with the math</h2>

<p>
  A prover $\mathcal{P}$ wants to convince a verifier $\mathcal{V}$ of a public statement $x$ without revealing the witness $w$. The classic structure is a three-move protocol:
</p>

<div class="math-block">
  <div class="math-label">Interactive Sigma Protocol</div>
  $$\mathcal{P} \xrightarrow{\text{ commitment } a} \mathcal{V} \xrightarrow{\text{ challenge } c \xleftarrow{\$} \mathbb{F}} \mathcal{P} \xrightarrow{\text{ response } z} \mathcal{V}$$
</div>

<p>
  Security rests on the verifier's challenge $c$ being unpredictable. But this requires the verifier to be online — useless for on-chain verification.
</p>

<p>
  The <strong>Fiat-Shamir heuristic</strong> (1986) removes this requirement by replacing the live challenge with a hash of the transcript. The prover computes:
</p>

<div class="math-block">
  <div class="math-label">Fiat-Shamir Transformation</div>
  $$c = H(\text{ctx} \| a)$$
</div>

<p>
  Here $H$ is a collision-resistant hash, $\text{ctx}$ is the public context (statement, protocol parameters, domain separators), and $a$ is the commitment. The challenge is now deterministic.
</p>

<p>
  In a multi-round protocol like WHIR, the transcript $\tau$ grows with each round:
</p>

<div class="math-block">
  <div class="math-label">Multi-Round Transcript</div>
  $$\tau_0 = H(\text{ctx})$$
  $$\tau_i = H(\tau_{i-1} \| a_i) \quad \text{for each round } i$$
  $$c_i = \tau_i$$
</div>

<p>
  The whole construction depends on one property: the hash must be <em>injective</em> over the sequence of absorbed values.
</p>

<div class="math-block">
  <div class="math-label">Injectivity Requirement</div>
  $$H(\tau \| \mathbf{v}) = H(\tau \| \mathbf{v}') \implies \mathbf{v} = \mathbf{v}'$$
</div>

<p>
  If two distinct sequences produce the same hash state, two different proofs generate identical challenges — the proof is no longer bound to a unique statement. This is <strong>transcript malleability</strong>, and it is exactly what H-4 introduces.
</p>

<!-- SPONGE -->
<h2 id="sponge">How the sponge absorbs scalars</h2>

<p>
  leanMultisig hashes with a Poseidon sponge. The sponge splits its internal state of $n$ field elements into two parts: the <strong>rate</strong> $r$ (the public absorption lane) and the <strong>capacity</strong> $c$ (the private security domain). Every time $r$ elements are absorbed, a permutation $\pi$ mixes them into the full state:
</p>

<div class="diagram-wrap">
<svg viewBox="0 0 680 220" xmlns="http://www.w3.org/2000/svg" style="width:100%;max-width:680px;display:block;margin:0 auto">
  <defs>
    <marker id="arr" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto">
      <path d="M0,0 L0,6 L8,3 z" fill="#555"/>
    </marker>
    <marker id="arr-g" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto">
      <path d="M0,0 L0,6 L8,3 z" fill="#1a7a4a"/>
    </marker>
  </defs>
  <rect x="20" y="80" width="180" height="60" rx="4" fill="#dbeafe" stroke="#3b82f6" stroke-width="1.5"/>
  <text x="110" y="107" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="12" fill="#1e40af" font-weight="600">RATE (r = 4 elements)</text>
  <text x="110" y="126" text-anchor="middle" font-family="'JetBrains Mono',monospace" font-size="11" fill="#3b82f6">s₀  s₁  s₂  s₃</text>
  <rect x="20" y="148" width="180" height="50" rx="4" fill="#f3e8ff" stroke="#9333ea" stroke-width="1.5"/>
  <text x="110" y="171" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="12" fill="#6b21a8" font-weight="600">CAPACITY (c)</text>
  <text x="110" y="188" text-anchor="middle" font-family="'JetBrains Mono',monospace" font-size="11" fill="#9333ea">s₄  s₅  …  sₙ</text>
  <text x="110" y="68" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="13" fill="#555" font-weight="600">Sponge State</text>
  <text x="250" y="55" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="12" fill="#555" font-weight="600">Input: [a, b, c]</text>
  <rect x="215" y="62" width="70" height="26" rx="3" fill="#fef3c7" stroke="#d97706" stroke-width="1"/>
  <text x="250" y="79" text-anchor="middle" font-family="'JetBrains Mono',monospace" font-size="12" fill="#92400e">a  b  c</text>
  <rect x="294" y="62" width="34" height="26" rx="3" fill="#fee2e2" stroke="#ef4444" stroke-width="1" stroke-dasharray="4,2"/>
  <text x="311" y="79" text-anchor="middle" font-family="'JetBrains Mono',monospace" font-size="12" fill="#dc2626">0</text>
  <text x="311" y="55" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="10" fill="#dc2626">padded!</text>
  <line x1="250" y1="90" x2="205" y2="100" stroke="#d97706" stroke-width="1.5" marker-end="url(#arr)"/>
  <circle cx="230" cy="110" r="10" fill="none" stroke="#555" stroke-width="1.5"/>
  <line x1="220" y1="110" x2="240" y2="110" stroke="#555" stroke-width="1.5"/>
  <line x1="230" y1="100" x2="230" y2="120" stroke="#555" stroke-width="1.5"/>
  <text x="230" y="138" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="10" fill="#555">XOR/add</text>
  <rect x="380" y="70" width="100" height="118" rx="6" fill="#f0fdf4" stroke="#16a34a" stroke-width="1.5"/>
  <text x="430" y="125" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="13" fill="#15803d" font-weight="600">π</text>
  <text x="430" y="143" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="11" fill="#15803d">Poseidon</text>
  <text x="430" y="158" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="11" fill="#15803d">Permutation</text>
  <line x1="202" y1="110" x2="378" y2="110" stroke="#555" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="290" y="105" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="10" fill="#555">absorb</text>
  <rect x="530" y="80" width="130" height="60" rx="4" fill="#dbeafe" stroke="#3b82f6" stroke-width="1.5"/>
  <text x="595" y="107" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="12" fill="#1e40af" font-weight="600">New Rate</text>
  <text x="595" y="124" text-anchor="middle" font-family="'JetBrains Mono',monospace" font-size="11" fill="#3b82f6">τ₀  τ₁  τ₂  τ₃</text>
  <rect x="530" y="148" width="130" height="50" rx="4" fill="#f3e8ff" stroke="#9333ea" stroke-width="1.5"/>
  <text x="595" y="178" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="11" fill="#6b21a8">New Capacity</text>
  <line x1="482" y1="110" x2="528" y2="110" stroke="#16a34a" stroke-width="1.5" marker-end="url(#arr-g)"/>
  <text x="595" y="67" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="12" fill="#555" font-weight="600">Challenge = τ₀</text>
  <line x1="595" y1="70" x2="595" y2="78" stroke="#555" stroke-width="1" stroke-dasharray="3,2"/>
</svg>
<div class="diagram-caption">
  Figure 1: One absorption round of the Poseidon sponge.
</div>
</div>

<!-- COLLISION DIAGRAM -->
<div class="diagram-wrap">
<svg viewBox="0 0 680 270" xmlns="http://www.w3.org/2000/svg" style="width:100%;max-width:680px;display:block;margin:0 auto">
  <defs>
    <marker id="arr2" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto">
      <path d="M0,0 L0,6 L8,3 z" fill="#555"/>
    </marker>
    <marker id="arr-r" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto">
      <path d="M0,0 L0,6 L8,3 z" fill="#dc2626"/>
    </marker>
  </defs>
  <text x="340" y="22" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="14" font-weight="700" fill="#111">The Collision: Two Different Inputs → Same State</text>
  <text x="170" y="55" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="13" fill="#1e40af" font-weight="600">Input A: [x₁, x₂, x₃]</text>
  <rect x="60" y="65" width="230" height="40" rx="4" fill="#dbeafe" stroke="#3b82f6" stroke-width="1.5"/>
  <text x="90" y="81" font-family="'JetBrains Mono',monospace" font-size="13" fill="#1e40af">x₁</text>
  <line x1="117" y1="65" x2="117" y2="105" stroke="#3b82f6" stroke-width="1"/>
  <text x="133" y="81" font-family="'JetBrains Mono',monospace" font-size="13" fill="#1e40af">x₂</text>
  <line x1="175" y1="65" x2="175" y2="105" stroke="#3b82f6" stroke-width="1"/>
  <text x="191" y="81" font-family="'JetBrains Mono',monospace" font-size="13" fill="#1e40af">x₃</text>
  <line x1="233" y1="65" x2="233" y2="105" stroke="#3b82f6" stroke-width="1"/>
  <rect x="233" y="65" width="57" height="40" rx="0" fill="#fee2e2" stroke="none"/>
  <text x="262" y="81" font-family="'JetBrains Mono',monospace" font-size="13" fill="#dc2626">0</text>
  <text x="262" y="97" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="9" fill="#dc2626">zero pad</text>
  <text x="170" y="155" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="13" fill="#16a34a" font-weight="600">Input B: [x₁, x₂, x₃, 0]</text>
  <rect x="60" y="165" width="230" height="40" rx="4" fill="#dcfce7" stroke="#16a34a" stroke-width="1.5"/>
  <text x="90" y="181" font-family="'JetBrains Mono',monospace" font-size="13" fill="#15803d">x₁</text>
  <line x1="117" y1="165" x2="117" y2="205" stroke="#16a34a" stroke-width="1"/>
  <text x="133" y="181" font-family="'JetBrains Mono',monospace" font-size="13" fill="#15803d">x₂</text>
  <line x1="175" y1="165" x2="175" y2="205" stroke="#16a34a" stroke-width="1"/>
  <text x="191" y="181" font-family="'JetBrains Mono',monospace" font-size="13" fill="#15803d">x₃</text>
  <line x1="233" y1="165" x2="233" y2="205" stroke="#16a34a" stroke-width="1"/>
  <text x="262" y="181" font-family="'JetBrains Mono',monospace" font-size="13" fill="#15803d">0</text>
  <line x1="292" y1="87" x2="388" y2="130" stroke="#3b82f6" stroke-width="2" marker-end="url(#arr2)"/>
  <line x1="292" y1="185" x2="388" y2="150" stroke="#16a34a" stroke-width="2" marker-end="url(#arr2)"/>
  <rect x="315" y="100" width="70" height="30" rx="4" fill="#fef9c3" stroke="#ca8a04" stroke-width="1.5"/>
  <text x="350" y="117" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="11" font-weight="700" fill="#92400e">IDENTICAL</text>
  <rect x="390" y="90" width="90" height="90" rx="8" fill="#f0f0f0" stroke="#888" stroke-width="1.5"/>
  <text x="435" y="133" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="18" fill="#333">π</text>
  <text x="435" y="152" text-anchor="middle" font-family="'Outfit',sans-serif" font-size="10" fill="#666">sponge</text>
  <line x1="482" y1="135" x2="540" y2="135" stroke="#dc2626" stroke-width="2.5" marker-end="url(#arr-r)"/>
  <rect x="542" y="110" width="120" height="50" rx="6" fill="#fee2e2" stroke="#dc2626" stroke-width="2"/>
  <text x="602" y="131" text-anchor="middle" font-family="'JetBrains Mono',monospace" font-size="12" fill="#dc2626" font-weight="500">SAME STATE</text>
  <text x="602" y="149" text-anchor="middle" font-family="'JetBrains Mono',monospace" font-size="11" fill="#991b1b">τ = τ'</text>
</svg>
<div class="diagram-caption">
  Figure 2: Both inputs pass through the same <code>observe_scalars</code> path. Input A has three elements; the fourth slot is zero-padded automatically by the chunking loop. Input B provides four elements, the last explicitly zero. The sponge sees identical buffers. No attacker-supplied zero is needed — the collision happens naturally whenever the input length is not a multiple of <code>RATE</code>.
</div>
</div>

<!-- H-4 -->
<h2 id="h4">H-4 — The collision bug that broke statement binding</h2>

<div class="finding-card high">
  <div class="finding-card-top">
    <span class="badge badge-high">🔴 High · Issue #171</span>
    <span class="badge badge-fixed">✓ Fixed in ee7377f</span>
  </div>
  <div class="finding-title">Transcript Collisions from Zero-Padded Scalar Absorption</div>
  <div class="finding-file">crates/backend/fiat-shamir/src/challenger.rs</div>
</div>

<div class="personal">
  This is the one I was most excited to find. It is not an edge-case crash. It is a violation of the core security property of the whole proof system.
</div>

<h3>The vulnerable code</h3>

<pre><code><span class="kw">pub fn</span> <span class="fn">observe_scalars</span>(&amp;<span class="kw">mut</span> self, scalars: &amp;[<span class="typ">F</span>]) {
    <span class="kw">for</span> chunk <span class="kw">in</span> scalars.<span class="fn">chunks</span>(RATE) {
        <span class="kw">let mut</span> buffer = [<span class="typ">F</span>::ZERO; RATE];
        <span class="kw">for</span> (i, val) <span class="kw">in</span> chunk.<span class="fn">iter</span>().<span class="fn">enumerate</span>() {
            buffer[i] = *val;
        }
        self.<span class="fn">observe</span>(buffer);
    }
}
</code></pre>

<p>
  The code is idiomatic Rust. It reads naturally. It is wrong in a subtle but devastating way.
</p>

<h3>Why this is a security violation</h3>

<p>
  When the input length is not a multiple of <code>RATE</code>, the last chunk is zero-padded. A proof for public input <code>[x₁, x₂, x₃]</code> absorbs <code>[x₁, x₂, x₃, 0]</code> into the sponge. A proof for <code>[x₁, x₂, x₃, 0]</code> — a different statement, four explicit elements — absorbs the same buffer. Both produce identical challenge sequences.
</p>

<p>
  The consequence: a proof for statement A is computationally valid for statement B whenever B's input is a zero-extended version of A's. On a multisig contract, an attacker could replay a valid approval for one signer set as if it were an approval for a padded superset — a completely different authorization.
</p>

<h3>The fix</h3>

<pre><code><span class="cmt">// prove_execution.rs</span>
prover_state.<span class="fn">add_base_scalars</span>(&amp;[
    vec![
        whir_config.starting_log_inv_rate,
        <span class="fn">log2_strict_usize</span>(memory.<span class="fn">len</span>()),
<span class="add">+       public_input.<span class="fn">len</span>(),</span>
    ],
    traces.<span class="fn">values</span>().<span class="fn">map</span>(|t| t.log_n_rows).<span class="fn">collect</span>(),
].<span class="fn">concat</span>());
</code></pre>

<p>
  Absorbing <code>public_input.len()</code> before the values makes this call-site injective — inputs of different lengths now hash to different states.
</p>

<div class="callout">
  <span class="icon">⚠️</span> <strong>Nuance:</strong> the fix is at the <em>call-site</em> in <code>prove_execution.rs</code>, not inside <code>observe_scalars</code>. The underlying API still zero-pads silently. Any future caller that omits the length prefix reintroduces the bug. A more defensive fix puts the length prefix inside <code>observe_scalars</code> so it cannot be forgotten.
</div>

<!-- M-1 -->
<h2 id="m1">M-1 — The bitmask that panics on 32-bit</h2>

<div class="finding-card medium">
  <div class="finding-card-top">
    <span class="badge badge-medium">🟡 Medium · Issue #172</span>
    <span class="badge badge-fixed">✓ Fixed in ee7377f</span>
  </div>
  <div class="finding-title">Bitmask Shift Can Panic on 32-Bit Targets</div>
  <div class="finding-file">crates/backend/fiat-shamir/src/challenger.rs</div>
</div>

<div class="personal">
  This one is sneakier. It would never show up in your CI on a standard x86-64 machine.
</div>

<h3>The vulnerable code</h3>

<pre><code><span class="kw">pub fn</span> <span class="fn">sample_in_range</span>(&amp;<span class="kw">mut</span> self, bits: <span class="typ">usize</span>, n_samples: <span class="typ">usize</span>) -&gt; <span class="typ">Vec</span>&lt;<span class="typ">usize</span>&gt; {
    <span class="fn">assert!</span>(bits &lt; <span class="typ">F</span>::<span class="fn">bits</span>());
    <span class="kw">let</span> sampled_fe = self.<span class="fn">sample_many</span>(n_samples.<span class="fn">div_ceil</span>(RATE)).<span class="fn">into_iter</span>().<span class="fn">flatten</span>();
    <span class="kw">let mut</span> res = <span class="typ">Vec</span>::new();
    <span class="kw">for</span> fe <span class="kw">in</span> sampled_fe.<span class="fn">take</span>(n_samples) {
        <span class="kw">let</span> rand_usize = fe.<span class="fn">as_canonical_u64</span>() <span class="kw">as</span> <span class="typ">usize</span>;
        res.<span class="fn">push</span>(rand_usize &amp; ((<span class="num">1</span> &lt;&lt; bits) - <span class="num">1</span>));
    }
    res
}
</code></pre>

<h3>Why this only breaks on 32-bit</h3>

<p>
  The literal <code>1</code> in <code>1 &lt;&lt; bits</code> is inferred as <code>usize</code>. On a 64-bit host every shift up to 63 succeeds. On a 32-bit target — WASM runtimes, embedded provers, some zkVM environments — <code>usize</code> is 32 bits, so any <code>bits &gt;= 32</code> overflows:
</p>

<ul style="font-family:'Outfit',sans-serif;font-size:16px;line-height:1.8;">
  <li>In <strong>debug builds</strong>: Rust panics with <em>"attempt to shift left with overflow"</em>.</li>
  <li>In <strong>release builds</strong>: the shift wraps silently, corrupting the bitmask with no error signal.</li>
</ul>

<p>
  The guard <code>assert!(bits &lt; F::bits())</code> checks against the <em>field</em> width — 64 for a 64-bit field. Values up to 63 pass. On 32-bit that is not enough.
</p>

<h3>The fix</h3>

<p>
  Shift in <code>u64</code> — always 64 bits wide — then cast:
</p>

<pre><code><span class="kw">let</span> mask = (<span class="num">1u64</span> &lt;&lt; bits) - <span class="num">1</span>;
<span class="kw">let</span> rand_usize = fe.<span class="fn">as_canonical_u64</span>() &amp; mask;
res.<span class="fn">push</span>(rand_usize <span class="kw">as</span> <span class="typ">usize</span>);
</code></pre>

<p>
  The existing assert guarantees <code>bits &lt;= 63</code>, so no additional guard is needed. One character — <code>1</code> becomes <code>1u64</code> — eliminates the entire class of 32-bit overflow risk.
</p>

<!-- M-2 -->
<h2 id="m2">M-2 — The overflow that turns PoW into a no-op</h2>

<div class="finding-card medium">
  <div class="finding-card-top">
    <span class="badge badge-medium">🟡 Medium · Issue #173</span>
    <span class="badge badge-fixed">✓ Fixed in ee7377f</span>
  </div>
  <div class="finding-title">Shift Overflow Lets PoW Grinding Check Be Bypassed</div>
  <div class="finding-file">crates/backend/fiat-shamir/src/verifier.rs</div>
</div>

<div class="personal">
  M-1 crashes or silently corrupts. M-2 is quieter and more dangerous — the verifier accepts every proof without complaint, at exactly the security level where you need it most.
</div>

<h3>The vulnerable code</h3>

<pre><code><span class="kw">if</span> self.challenger.state[<span class="num">0</span>].<span class="fn">as_canonical_u64</span>() &amp; ((<span class="num">1</span> &lt;&lt; bits) - <span class="num">1</span>) != <span class="num">0</span> {
    <span class="kw">return</span> <span class="typ">Err</span>(<span class="typ">ProofError</span>::InvalidGrindingWitness);
}
</code></pre>

<p>
  At first glance this looks correct: verify that the leading <code>bits</code> of the state are zero, reject otherwise. But an integer edge case turns the whole check into dead code.
</p>

<h3>How <code>bits = 64</code> is reached</h3>

<p>
  <code>bits</code> is read directly from the WHIR configuration. At maximum security the field is 64-bit, so the code sets <code>bits = F::bits()</code> — exactly 64. This is not a caller mistake; it is the intended value at the highest grinding difficulty.
</p>

<div class="callout danger">
  <span class="icon">⚠️</span> When <code>bits = 64</code>, the expression <code>(1u64 &lt;&lt; 64) - 1</code> wraps to <code>0</code> in Rust release mode. The bitmask is <code>0</code>, so <code>state[0] &amp; 0 == 0</code> is always <code>true</code>, the inequality <code>!= 0</code> is always <code>false</code>, and the check <em>never</em> rejects. Any nonce passes. The grinding requirement — the main cost imposed on a WHIR prover to deter spam — is completely bypassed at the very difficulty level where it matters most.
</div>

<h3>The fix</h3>

<p>
  Guard the shift explicitly so that <code>bits = 64</code> produces the correct all-ones mask instead of wrapping to zero:
</p>

<pre><code><span class="kw">let</span> mask: <span class="typ">u64</span> = <span class="kw">if</span> bits &gt;= <span class="num">64</span> {
    <span class="typ">u64</span>::MAX
} <span class="kw">else</span> {
    (<span class="num">1u64</span> &lt;&lt; bits) - <span class="num">1</span>
};
<span class="kw">if</span> self.challenger.state[<span class="num">0</span>].<span class="fn">as_canonical_u64</span>() &amp; mask != <span class="num">0</span> {
    <span class="kw">return</span> <span class="typ">Err</span>(<span class="typ">ProofError</span>::InvalidGrindingWitness);
}
</code></pre>

<!-- LESSONS -->
<h2 id="lessons">What I learned — and what you should check in your own code</h2>

<p>
  None of these bugs reflect carelessness. All three are invisible to standard testing: H-4 requires two crafted inputs, M-1 only fires on non-x86-64 targets, M-2 only strikes at an exact parameter boundary. The lesson is not to test harder — it is to build APIs that make the mistakes structurally impossible.
</p>

<h3>1. The Fiat-Shamir injectivity checklist</h3>

<div class="callout">
  <strong>✓</strong> Are lengths encoded before data? <br/>
  <strong>✓</strong> Are domain separators used? <br/>
  <strong>✓</strong> Can two different inputs reach the same transcript state?
</div>

<p>
  On domain separators: <code>leanMultisig</code> initialises the sponge with a fixed protocol tag, which blocks cross-protocol replay. But the tag is a compile-time constant — it is not parameterised by circuit shape or WHIR folding configuration. Two proofs with different circuit sizes but the same field width can share an identical opening prefix. The separator works at the protocol boundary; it does not work at the instance level. That is a separate, lower-severity issue, but worth auditing in any fork.
</p>

<h3>2. Encode widths explicitly — never trust type inference for security arithmetic</h3>

<p>
  M-1 and M-2 are the same bug in different clothes: a shift on an integer whose width is platform-inferred. The fix is identical in both cases: write the type. <code>1u64</code>, not <code>1</code>. Any arithmetic touching security parameters — bit widths, difficulty targets, field sizes — should use fixed-width types. The compiler will not warn you when platform-dependent sizes collide with security boundaries.
</p>

<h3>3. Test at the exact documented limit, not just below it</h3>

<p>
  M-2 was only reachable at <code>bits = F::bits()</code> — the maximum. A test at <code>bits = 63</code> passes. A test at <code>bits = 64</code> catches the bug immediately. Whenever a function accepts a parameter with a documented maximum, add a test at that exact value. This applies to any bit-count, difficulty level, or field-size-derived input.
</p>

<hr/>

<h2 id="summary">Summary</h2>

<table>
  <thead>
    <tr>
      <th>ID</th>
      <th>Finding</th>
      <th>Severity</th>
      <th>Status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>H-4</code></td>
      <td>Transcript collision via zero-pad absorption</td>
      <td><span class="badge badge-high">High</span></td>
      <td><span class="badge badge-fixed">Fixed</span></td>
    </tr>
    <tr>
      <td><code>M-1</code></td>
      <td>Bitmask shift panic on 32-bit</td>
      <td><span class="badge badge-medium">Medium</span></td>
      <td><span class="badge badge-fixed">Fixed</span></td>
    </tr>
    <tr>
      <td><code>M-2</code></td>
      <td>PoW grinding bypass via shift overflow</td>
      <td><span class="badge badge-medium">Medium</span></td>
      <td><span class="badge badge-fixed">Fixed</span></td>
    </tr>
  </tbody>
</table>

<p style="color:var(--muted); font-size:15px; margin-top: 40px;">
  Issues: <a href="https://github.com/leanEthereum/leanMultisig/issues/171">#171</a>, <a href="https://github.com/leanEthereum/leanMultisig/issues/172">#172</a>, <a href="https://github.com/leanEthereum/leanMultisig/issues/173">#173</a> <br/>
  Fix: <a href="https://github.com/leanEthereum/leanMultisig/commit/ee7377f9ef5a39cbbd67662c14c2ad6100494426">ee7377f</a> · Author: <a href="https://github.com/this-vishalsingh">@this-vishalsingh</a>
</p>
