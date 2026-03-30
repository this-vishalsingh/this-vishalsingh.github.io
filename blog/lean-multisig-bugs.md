---
layout: default
title: I Found Three Bugs in leanMultisig's Fiat-Shamir Backend
permalink: /blog/lean-multisig-bugs/
active: blog
---

<script>
  window.MathJax = {
    tex: { inlineMath: [['$','$'], ['\\(','\\)']], displayMath: [['$$','$$']] },
    options: { skipHtmlTags: ['script','noscript','style','textarea'] }
  };
</script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/3.2.2/es5/tex-chtml.min.js"></script>

<style>
/* Scoped styles for high-fidelity post */
.hf-post {
  --ink:     #1a1c20;
  --page:    #f7f4ef;
  --warm:    #ede8e0;
  --border:  #d4cfc6;
  --accent:  #c0392b;
  --accent2: #2c7bb5;
  --green:   #1a7a4a;
  --muted:   #7a756e;
  --bright:  #0f1115;
  --code-bg: #1e2128;
  --code-txt:#d4d8e0;
  
  background: var(--page);
  color: var(--ink);
  font-family: 'Lora', serif;
  font-size: 18px;
  line-height: 1.8;
  margin: -60px -80px; /* Counter-act main_content padding */
  border-radius: 4px;
  overflow: hidden;
}

/* Override site container for this specific post */
#main_content { 
  max-width: 800px !important; 
  padding: 60px 80px !important;
  background: white !important; /* Base background */
}

@media(max-width: 768px) {
  #main_content { padding: 30px 25px !important; }
  .hf-post { margin: -30px -25px; }
}

.hf-post .hero {
  background: var(--ink);
  color: #eef0f4;
  padding: 72px 40px 64px;
  position: relative;
  overflow: hidden;
}
.hf-post .hero::before {
  content: '';
  position: absolute;
  inset: 0;
  background: repeating-linear-gradient(
    0deg, transparent, transparent 39px,
    rgba(255,255,255,0.03) 39px, rgba(255,255,255,0.03) 40px
  );
  pointer-events: none;
}
.hf-post .hero-inner { max-width: 760px; margin: 0 auto; position: relative; }

.hf-post .hero-tags {
  display: flex; gap: 8px; flex-wrap: wrap; margin-bottom: 28px;
}
.hf-post .htag {
  font-family: 'JetBrains Mono', monospace;
  font-size: 10px;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  padding: 4px 10px;
  border-radius: 3px;
}
.hf-post .htag-red    { background: rgba(192,57,43,0.3);  color: #ff8a7a; border: 1px solid rgba(192,57,43,0.5); }
.hf-post .htag-blue   { background: rgba(44,123,181,0.2); color: #7ab8e8; border: 1px solid rgba(44,123,181,0.4); }
.hf-post .htag-yellow { background: rgba(232,255,107,0.12); color: #e8ff6b; border: 1px solid rgba(232,255,107,0.3); }

.hf-post .hero h1 {
  font-family: 'Lora', serif;
  font-size: clamp(28px, 4.5vw, 46px);
  line-height: 1.2;
  font-weight: 600;
  margin-bottom: 24px;
  color: #f0ece4;
  border: none !important;
}
.hf-post .hero .lede {
  font-family: 'Outfit', sans-serif;
  font-size: 17px;
  color: #9a9590;
  line-height: 1.65;
  max-width: 640px;
  margin-bottom: 36px;
  font-weight: 300;
}
.hf-post .hero-meta {
  display: flex;
  align-items: center;
  gap: 20px;
  font-family: 'Outfit', sans-serif;
  font-size: 13px;
  color: #666;
  flex-wrap: wrap;
}
.hf-post .hero-meta .ava {
  width: 34px; height: 34px; border-radius: 50%;
  background: linear-gradient(135deg, #e8ff6b, #6bffe8);
  display: flex; align-items: center; justify-content: center;
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px; font-weight: 500; color: #000;
  flex-shrink: 0;
}
.hf-post .hero-meta a { color: #7ab8e8 !important; text-decoration: none !important; border-bottom: none !important; }
.hf-post .hero-meta a:hover { text-decoration: underline !important; }

/* LAYOUT */
.hf-post .container {
  max-width: 760px;
  margin: 0 auto;
  padding: 0 40px;
}

/* TOC */
.hf-post .toc-wrap {
  max-width: 760px; margin: 48px auto; padding: 0 40px;
}
.hf-post .toc {
  background: var(--warm);
  border: 1px solid var(--border);
  border-radius: 6px;
  padding: 24px 28px;
}
.hf-post .toc-head {
  font-family: 'Outfit', sans-serif;
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: var(--muted);
  margin-bottom: 16px;
}
.hf-post .toc ol { padding-left: 20px; }
.hf-post .toc li { margin-bottom: 8px; font-family: 'Outfit', sans-serif; font-size: 15px; }
.hf-post .toc a { color: var(--accent2); text-decoration: none; border-bottom: none !important; }
.hf-post .toc a:hover { text-decoration: underline !important; }

/* TYPOGRAPHY */
.hf-post article { padding: 12px 0 100px; }

.hf-post h2 {
  font-family: 'Lora', serif !important;
  font-size: 28px !important;
  font-weight: 600 !important;
  color: var(--bright) !important;
  margin: 60px 0 20px !important;
  line-height: 1.3 !important;
  border-bottom: 2px solid var(--border) !important;
  padding-bottom: 12px !important;
  background: none !important;
}

.hf-post h3 {
  font-family: 'Outfit', sans-serif !important;
  font-size: 16px !important;
  font-weight: 600 !important;
  color: var(--bright) !important;
  margin: 32px 0 10px !important;
  text-transform: uppercase !important;
  letter-spacing: 0.06em !important;
  border: none !important;
}

.hf-post p { margin-bottom: 20px !important; }
.hf-post a { color: var(--accent2) !important; text-decoration: none !important; }
.hf-post a:hover { border-bottom: 1px solid var(--accent2) !important; }

.hf-post strong { font-weight: 600; color: var(--bright); }
.hf-post em { font-style: italic; color: var(--muted); }

/* PERSONAL NOTE */
.hf-post .personal {
  background: #fff;
  border-left: 4px solid var(--accent2);
  padding: 20px 24px;
  border-radius: 0 6px 6px 0;
  margin: 28px 0;
  font-style: italic;
  color: #444;
  font-size: 16.5px;
}
.hf-post .personal::before {
  content: '💬  ';
  font-style: normal;
}

/* CALLOUT */
.hf-post .callout {
  background: var(--warm);
  border: 1px solid var(--border);
  border-radius: 6px;
  padding: 20px 24px;
  margin: 24px 0;
  font-family: 'Outfit', sans-serif;
  font-size: 15.5px;
  line-height: 1.65;
}
.hf-post .callout.danger {
  background: rgba(192,57,43,0.07);
  border-color: rgba(192,57,43,0.3);
}
.hf-post .callout.success {
  background: rgba(26,122,74,0.07);
  border-color: rgba(26,122,74,0.3);
}
.hf-post .callout .icon { font-size: 18px; margin-right: 6px; }

/* FINDING CARD */
.hf-post .finding-card {
  border-radius: 8px;
  padding: 24px 28px;
  margin: 32px 0;
  border: 1px solid;
}
.hf-post .finding-card.high {
  background: rgba(192,57,43,0.06);
  border-color: rgba(192,57,43,0.25);
}
.hf-post .finding-card.medium {
  background: rgba(230,126,34,0.06);
  border-color: rgba(230,126,34,0.25);
}
.hf-post .finding-card-top {
  display: flex; align-items: center; justify-content: space-between;
  margin-bottom: 10px; flex-wrap: wrap; gap: 8px;
}
.hf-post .badge {
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  padding: 3px 10px;
  border-radius: 99px;
  letter-spacing: 0.05em;
}
.hf-post .badge-high   { background: rgba(192,57,43,0.2); color: #c0392b; }
.hf-post .badge-medium { background: rgba(230,126,34,0.2); color: #e67e22; }
.hf-post .badge-fixed  { background: rgba(26,122,74,0.2); color: #1a7a4a; }
.hf-post .finding-title {
  font-family: 'Outfit', sans-serif;
  font-size: 17px;
  font-weight: 600;
  color: var(--bright);
  margin-bottom: 4px;
}
.hf-post .finding-file {
  font-family: 'JetBrains Mono', monospace;
  font-size: 12px;
  color: var(--muted);
}

/* CODE */
.hf-post pre {
  background: var(--code-bg);
  color: var(--code-txt);
  border-radius: 8px;
  padding: 22px 24px;
  overflow-x: auto;
  margin: 20px 0;
  font-family: 'JetBrains Mono', monospace;
  font-size: 13.5px;
  line-height: 1.7;
  border: 1px solid #2e333d;
}
.hf-post code {
  font-family: 'JetBrains Mono', monospace;
  font-size: 14px;
  background: rgba(0,0,0,0.08);
  padding: 2px 6px;
  border-radius: 4px;
  color: #333;
}
.hf-post pre code { background: none; padding: 0; color: inherit; font-size: inherit; }
.hf-post .kw  { color: #c792ea; }
.hf-post .fn  { color: #82aaff; }
.hf-post .cmt { color: #546e7a; font-style: italic; }
.hf-post .typ { color: #ffcb6b; }
.hf-post .num { color: #f78c6c; }
.hf-post .str { color: #c3e88d; }
.hf-post .del { display: block; background: rgba(192,57,43,0.15); color: #ff8a7a; }
.hf-post .add { display: block; background: rgba(26,122,74,0.15); color: #a3e4b5; }

/* MATH DISPLAY */
.hf-post .math-block {
  background: #fff;
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 24px 28px;
  margin: 24px 0;
  text-align: center;
  overflow-x: auto;
  color: var(--ink);
}
.hf-post .math-label {
  font-family: 'Outfit', sans-serif;
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: var(--muted);
  margin-bottom: 14px;
}

/* SVG DIAGRAMS */
.hf-post .diagram-wrap {
  margin: 32px 0;
  background: #fff;
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 24px;
  overflow-x: auto;
}
.hf-post .diagram-caption {
  font-family: 'Outfit', sans-serif;
  font-size: 13px;
  color: var(--muted);
  text-align: center;
  margin-top: 14px;
  font-style: italic;
}

/* SUMMARY TABLE */
.hf-post table {
  width: 100% !important;
  border-collapse: collapse !important;
  font-family: 'Outfit', sans-serif !important;
  font-size: 14.5px !important;
  margin: 28px 0 !important;
}
.hf-post th {
  text-align: left !important;
  padding: 10px 14px !important;
  font-size: 11px !important;
  text-transform: uppercase !important;
  letter-spacing: 0.08em !important;
  color: var(--muted) !important;
  border-bottom: 2px solid var(--border) !important;
  border: none !important;
  background: transparent !important;
}
.hf-post td {
  padding: 14px !important;
  border-bottom: 1px solid var(--border) !important;
  vertical-align: top !important;
  line-height: 1.5 !important;
  border-left: none !important;
  border-right: none !important;
  border-top: none !important;
}
.hf-post tr:last-child td { border-bottom: none !important; }
</style>

<div class="hf-post">

<!-- HERO -->
<div class="hero">
  <div class="hero-inner">
    <div class="hero-tags">
      <span class="htag htag-red">H-4 · High</span>
      <span class="htag htag-blue">M-1 · Medium</span>
      <span class="htag htag-blue">M-2 · Medium</span>
      <span class="htag htag-yellow">Fiat-Shamir</span>
      <span class="htag htag-yellow">ZK Security</span>
    </div>

    <h1>I Found Three Bugs in leanMultisig's Fiat-Shamir Backend</h1>

    <p class="lede">
      One high-severity transcript collision that could let an attacker recycle proofs.
      Two silent overflow bombs hiding inside the same 300-line module.
      Here is exactly what I found, why it matters, and how to think about it.
    </p>

    <div class="hero-meta">
      <div class="ava">VS</div>
      <div>
        <strong style="color:#d4d8e0">Vishal Singh</strong> &nbsp;·&nbsp;
        <a href="https://github.com/this-vishalsingh">@this-vishalsingh</a>
      </div>
      <span>·</span>
      <span>March 28, 2026</span>
      <span>·</span>
      <a href="https://github.com/leanEthereum/leanMultisig/commit/ee7377f9ef5a39cbbd67662c14c2ad6100494426">Fix: commit ee7377f ↗</a>
    </div>
  </div>
</div>

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
    </ol>
  </div>
</div>

<!-- ARTICLE -->
<div class="container">
<article>

<!-- STORY -->
<h2 id="story">How I got here</h2>

<p>
  I had been reading through the leanEthereum codebase for a few days, not looking for anything in particular — just trying to understand how the proving pipeline works end-to-end. The high-level circuits are beautiful. The Lean 4 formalization is genuinely impressive. But the more I read, the more I kept coming back to one folder: <code>crates/backend/fiat-shamir/</code>.
</p>

<div class="personal">
  Every time I look at a ZK system and want to find a bug, I start with the transcript. Not the circuits. Not the constraint system. The transcript. Because circuits can be perfectly specified and formally verified, and the whole system still falls apart if the randomness feeding into it is not unique. The transcript is the one place where a subtle mistake becomes a catastrophic one.
</div>

<p>
  I spent about a day reading <code>challenger.rs</code> and <code>verifier.rs</code>. What I found were three bugs — different in severity, but all rooted in the same underlying assumption: that the caller guarantees well-formed inputs. In a proof system, that is the assumption you should <em>never</em> make.
</p>

<p>
  Let me walk you through all of them. But first — some math.
</p>

<!-- FIAT-SHAMIR PRIMER -->
<h2 id="what-is-fs">What Fiat-Shamir actually is — with the math</h2>

<p>
  Imagine a protocol where a prover $\mathcal{P}$ wants to convince a verifier $\mathcal{V}$ that they know some secret witness $w$ for a public statement $x$, without revealing $w$. The classic structure is a three-move protocol:
</p>

<div class="math-block">
  <div class="math-label">Interactive Sigma Protocol</div>
  $$\mathcal{P} \xrightarrow{\text{ commitment } a} \mathcal{V} \xrightarrow{\text{ challenge } c \xleftarrow{\$} \mathbb{F}} \mathcal{P} \xrightarrow{\text{ response } z} \mathcal{V}$$
</div>

<p>
  The security relies on the verifier sending a <em>random</em> challenge $c$ that the prover cannot predict in advance. But this is interactive — the verifier has to be online and participate. That is useless for on-chain verification.
</p>

<p>
  The <strong>Fiat-Shamir heuristic</strong>, introduced in 1986, replaces the verifier's random challenge with a hash of the transcript so far. The prover computes:
</p>

<div class="math-block">
  <div class="math-label">Fiat-Shamir Transformation</div>
  $$c = H(\text{ctx} \| a)$$
</div>

<p>
  where $H$ is a collision-resistant hash function (or a random oracle in the security proof), $\text{ctx}$ is the public context (the statement $x$, protocol parameters, domain separators), and $a$ is the prover's commitment. The challenge is now deterministic and non-interactive.
</p>

<p>
  In a multi-round protocol like WHIR, this generalises to a transcript $\tau$ that grows with each round:
</p>

<div class="math-block">
  <div class="math-label">Multi-Round Transcript</div>
  $$\tau_0 = H(\text{ctx})$$
  $$\tau_i = H(\tau_{i-1} \| a_i) \quad \text{for each round } i$$
  $$c_i = \tau_i$$
</div>

<p>
  The security of this construction rests on one critical property. The hash must be <em>injective</em> with respect to the sequence of absorbed values. Formally:
</p>

<div class="math-block">
  <div class="math-label">Injectivity Requirement</div>
  $$H(\tau \| \mathbf{v}) = H(\tau \| \mathbf{v}') \implies \mathbf{v} = \mathbf{v}'$$
</div>

<p>
  If two different sequences $\mathbf{v} \neq \mathbf{v}'$ can produce the same hash state, then two different proofs produce the same challenges, and the proof is no longer bound to a unique statement. That is the definition of <strong>transcript malleability</strong>, and it is exactly what bug H-4 introduces.
</p>

<!-- SPONGE -->
<h2 id="sponge">How the sponge absorbs scalars</h2>

<p>
  leanMultisig uses a Poseidon-based sponge construction as its hash function. Before we look at the bug, let us understand how the sponge works normally.
</p>

<p>
  A sponge has an internal state of $n$ field elements. It is split into two parts: the <strong>rate</strong> $r$ (the public part that absorbs input) and the <strong>capacity</strong> $c$ (the private part that provides security). Each time we absorb $r$ elements, we apply a permutation $\pi$ to mix them into the state:
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
  Figure 2: Input A and Input B produce <em>identical</em> sponge buffers after chunk-padding.
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
  The code is idiomatic Rust. It reads naturally. And it is wrong in a subtle but devastating way.
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
  By absorbing <code>public_input.len()</code> into the transcript <em>before</em> the values, the construction becomes injective again.
</p>

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

<h3>The vulnerable code</h3>

<pre><code><span class="kw">if</span> self.challenger.state[<span class="num">0</span>].<span class="fn">as_canonical_u64</span>() &amp; ((<span class="num">1</span> &lt;&lt; bits) - <span class="num">1</span>) != <span class="num">0</span> {
    <span class="kw">return</span> <span class="typ">Err</span>(<span class="typ">ProofError</span>::InvalidGrindingWitness);
}
</code></pre>

<div class="callout danger">
  <span class="icon">⚠️</span> The grinding check silently accepts <em>any</em> proof when <code>bits = 64</code> because the mask becomes zero.
</div>

<!-- LESSONS -->
<h2 id="lessons">What I learned — and what you should check in your own code</h2>

<p>
  Finding these bugs does not mean the team was careless. All three are genuinely subtle.
</p>

<h3>1. The Fiat-Shamir injectivity checklist</h3>

<div class="callout">
  <strong>✓</strong> Are lengths encoded before data? <br/>
  <strong>✓</strong> Are domain separators used? <br/>
  <strong>✓</strong> Can two different inputs reach the same transcript state?
</div>

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

</article>
</div>

</div>
