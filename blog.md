---
layout: default
title: Blogs
active: blog
permalink: /blog/
---

<style>
/* Organic Tag styles tailored for light theme */
.blog-tags-filter {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 2rem;
}

.blog-tag-btn {
  font-family: 'Inter', sans-serif;
  padding: 6px 14px;
  font-size: 13px;
  font-weight: 500;
  border-radius: 6px;
  background: #f9f9f9;
  border: 1px solid #ddd;
  color: #555;
  cursor: pointer;
  transition: all 0.2s ease;
}

.blog-tag-btn:hover {
  background: #fff;
  border-color: #1e6bb8;
  color: #111;
}

.blog-tag-btn.active {
  background: #1e6bb8;
  color: #fff;
  border-color: #1e6bb8;
}

.article-tags {
  display: flex;
  gap: 8px;
  margin-bottom: 12px;
}

.tag-badge {
  font-family: 'Inter', sans-serif;
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  padding: 3px 8px;
  border-radius: 4px;
  background: #f4f4f4;
  color: #666;
  border: 1px solid #eee;
}

/* Specific Tag Colors for Light Theme */
.tag-badge.zk { color: #5a228b; background: #faf5ff; border-color: #e9d5ff; }
.tag-badge.vuln { color: #b91c1c; background: #fef2f2; border-color: #fecaca; }
.tag-badge.audit { color: #15803d; background: #f0fdf4; border-color: #bbf7d0; }
.tag-badge.cairo { color: #0369a1; background: #f0f9ff; border-color: #bae6fd; }
.tag-badge.fhe { color: #c2410c; background: #fff7ed; border-color: #fed7aa; }

.blog-no-results {
  display: none;
  text-align: center;
  color: #666;
  padding: 3rem 0;
  font-family: 'Inter', sans-serif;
  font-size: 15px;
}

.blog-no-results.visible {
  display: block;
}

.blog-card.hidden {
  display: none !important;
}
</style>

# In the mind of thisvishalsingh

Join me as I explore the complex and fascinating world of Ethereum and other Blockchains. I look under the hood of protocols, Layer 2 solutions, and developer tools. From technical tutorials to in-depth cryptography research and DeFi analyses, there's much to explore.

<div class="search-section" style="margin-top: 1.5rem;">
  <div class="search-bar-wrapper">
    <span class="search-icon">🔍</span>
    <input type="text" id="blogSearch" placeholder="Search posts…" oninput="filterBlogCards()" autocomplete="off">
  </div>
</div>

<div class="blog-tags-filter" id="blogTagFilters">
  <button class="blog-tag-btn active" onclick="setTagFilter('all', this)">All</button>
  <button class="blog-tag-btn" onclick="setTagFilter('zk', this)">ZK Proofs</button>
  <button class="blog-tag-btn" onclick="setTagFilter('audit', this)">Audit</button>
  <button class="blog-tag-btn" onclick="setTagFilter('cairo', this)">Cairo</button>
  <button class="blog-tag-btn" onclick="setTagFilter('fhe', this)">FHE</button>
  <button class="blog-tag-btn" onclick="setTagFilter('vuln', this)">Vulnerabilities</button>
</div>

<div class="paper-list" id="blogGrid">

  <!-- GOAT Network -->
  <a href="/blog/goat-slash-fraction-bypass/" class="paper-card blog-card" data-tags="vuln audit" data-title="GOAT Network — Slash Fraction Validation Bypass">
    <div class="article-tags">
      <span class="tag-badge vuln">Vulnerability</span>
      <span class="tag-badge audit">Audit</span>
    </div>
    <span class="paper-title">GOAT Network — Slash Fraction Validation Bypass</span>
    <p class="paper-excerpt">A logic flaw in GOAT Network's slash fraction calculations allowing negative slashing, validator inflation, and token siphoning.</p>
    <div style="margin-top: 12px; display: block;">
      <span class="paper-meta" style="display: inline-block; margin-bottom: 0;">Apr 03, 2026</span>
    </div>
  </a>

  <!-- LeanMultisig -->
  <a href="/blog/lean-multisig-bugs/" class="paper-card blog-card" data-tags="zk vuln" data-title="Breaking LeanMultisig zkVM Shamir (Part 1)">
    <div class="article-tags">
      <span class="tag-badge zk">ZK</span>
      <span class="tag-badge vuln">Vulnerability</span>
    </div>
    <span class="paper-title">Breaking LeanMultisig zkVM Shamir (Part 1)</span>
    <p class="paper-excerpt">Three bugs in LeanMultisig's Fiat-Shamir layer — transcript malleability, 32-bit shift panic, and a PoW bypass at max difficulty. All share the same root cause.</p>
    <div style="margin-top: 12px; display: block;">
      <span class="paper-meta" style="display: inline-block; margin-bottom: 0;">Mar 28, 2026</span>
    </div>
  </a>

  <!-- Cairo Security Checklist -->
  <a href="/blog/cairo-security-checklist/" class="paper-card blog-card" data-tags="cairo vuln audit" data-title="Cairo Security Vulnerabilities Checklist">
    <div class="article-tags">
      <span class="tag-badge cairo">Cairo</span>
      <span class="tag-badge vuln">Vulnerability</span>
      <span class="tag-badge audit">Audit</span>
    </div>
    <span class="paper-title">Cairo Security Vulnerabilities Checklist</span>
    <p class="paper-excerpt">A comprehensive checklist of common security pitfalls and vulnerability patterns in Cairo smart contracts on StarkNet.</p>
    <div style="margin-top: 12px; display: block;">
      <span class="paper-meta" style="display: inline-block; margin-bottom: 0;">2026</span>
    </div>
  </a>

  <!-- ZK-SNARKs Security -->
  <a href="/blog/zksnarks-security" class="paper-card blog-card" data-tags="zk vuln" data-title="ZK-SNARKs Security - Vulnerabilities and Root Causes">
    <div class="article-tags">
      <span class="tag-badge zk">ZK</span>
      <span class="tag-badge vuln">Vulnerability</span>
    </div>
    <span class="paper-title">ZK-SNARKs Security — Vulnerabilities and Root Causes</span>
    <p class="paper-excerpt">An analysis of the most critical vulnerability classes in ZK-SNARK systems, their root causes, and how to reason about them during an audit.</p>
    <div style="margin-top: 12px; display: block;">
      <span class="paper-meta" style="display: inline-block; margin-bottom: 0;">2026</span>
    </div>
  </a>

  <!-- Nova and CycleFold -->
  <a href="/blog/nova-cyclefold/" class="paper-card blog-card" data-tags="zk" data-title="Nova and CycleFold">
    <div class="article-tags">
      <span class="tag-badge zk">ZK</span>
    </div>
    <span class="paper-title">Nova and CycleFold</span>
    <p class="paper-excerpt">A deep dive into Nova's recursive proof system and CycleFold's cycle-of-curves folding scheme — how they work, their security assumptions, and trade-offs.</p>
    <div style="margin-top: 12px; display: block;">
      <span class="paper-meta" style="display: inline-block; margin-bottom: 0;">2026</span>
    </div>
  </a>

  <!-- FHE Audit Checklist -->
  <a href="/blog/fhe-audit-checklist/" class="paper-card blog-card" data-tags="fhe audit" data-title="FHE Protocol Security Audit Checklist">
    <div class="article-tags">
      <span class="tag-badge fhe">FHE</span>
      <span class="tag-badge audit">Audit</span>
    </div>
    <span class="paper-title">FHE Protocol Security Audit Checklist</span>
    <p class="paper-excerpt">A structured checklist for auditing Fully Homomorphic Encryption protocols — covering parameter selection, noise budgets, circuit correctness, and implementation pitfalls.</p>
    <div style="margin-top: 12px; display: block;">
      <span class="paper-meta" style="display: inline-block; margin-bottom: 0;">2026</span>
    </div>
  </a>

  <!-- Audit Process -->
  <a href="/blog/audit-process/" class="paper-card blog-card" data-tags="audit" data-title="My Security Audit Process">
    <div class="article-tags">
      <span class="tag-badge audit">Audit</span>
    </div>
    <span class="paper-title">My Security Audit Process</span>
    <p class="paper-excerpt">A transparent look at how I work with clients — from scope definition and kick-off calls through multi-pass code review, reporting, and fix verification.</p>
    <div style="margin-top: 12px; display: block;">
      <span class="paper-meta" style="display: inline-block; margin-bottom: 0;">2026</span>
    </div>
  </a>

  <div class="blog-no-results" id="blogNoResults">No posts match your search.</div>

</div>

<script>
var activeTag = 'all';

function filterBlogCards() {
  var query = document.getElementById('blogSearch').value.toLowerCase().trim();
  var cards  = document.querySelectorAll('.blog-card');
  var shown  = 0;

  cards.forEach(function(card) {
    var title = (card.dataset.title || '').toLowerCase();
    var tags  = (card.dataset.tags  || '').toLowerCase();
    var matchQ = !query || title.includes(query) || tags.includes(query);
    var matchT = activeTag === 'all' || tags.includes(activeTag);

    if (matchQ && matchT) {
      card.classList.remove('hidden');
      shown++;
    } else {
      card.classList.add('hidden');
    }
  });

  var noResults = document.getElementById('blogNoResults');
  noResults.classList.toggle('visible', shown === 0);
}

function setTagFilter(tag, btn) {
  activeTag = tag;
  document.querySelectorAll('.blog-tag-btn').forEach(function(b) {
    b.classList.remove('active');
  });
  btn.classList.add('active');
  filterBlogCards();
}
</script>
