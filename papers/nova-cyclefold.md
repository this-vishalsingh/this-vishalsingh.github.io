---
layout: default
title: Design Review of Nova + CycleFold Implementation
permalink: /papers/nova-cyclefold/
active: papers
---

# Design Review of Nova + CycleFold Implementation

<div class="tldr-box">
  <strong>TL;DR</strong>
  Nova turns "prove a long computation" into "prove one step and fold it into a running claim," using a folding scheme for committed relaxed R1CS. CycleFold is a clean way to use a cycle of curves so that (most of) those elliptic-curve costs don't bloat the recursive verifier circuit.
</div>

This research paper provides a deep-dive into the architectural nuances and security considerations of combining Nova's folding proof system with the CycleFold optimization.

---

### Full Paper

<div class="pdf-viewer-wrapper">
  <div class="pdf-viewer-toolbar">
    <span class="pdf-viewer-label">📄 Design Review of Nova + CycleFold Implementation</span>
    <div class="pdf-viewer-actions">
      <a href="https://raw.githubusercontent.com/this-vishalsingh/publications/main/research/draft-Design%20Review%20of%20Nova%20%2B%20CycleFold%20Implementation.pdf" download class="pdf-action-btn" title="Download PDF">
        ↓ Download
      </a>
      <a href="https://github.com/this-vishalsingh/publications/blob/main/research/draft-Design%20Review%20of%20Nova%20%2B%20CycleFold%20Implementation.pdf" target="_blank" rel="noopener" class="pdf-action-btn pdf-action-btn--ghost" title="View on GitHub">
        ↗ GitHub
      </a>
    </div>
  </div>
  <div class="pdf-viewer-frame-wrap">
    <div class="pdf-loading-indicator" id="pdfLoader">
      <div class="pdf-spinner"></div>
      <span>Loading paper…</span>
    </div>
    <iframe
      id="pdfFrame"
      class="pdf-viewer-frame"
      src="https://docs.google.com/viewer?url=https%3A%2F%2Fraw.githubusercontent.com%2Fthis-vishalsingh%2Fpublications%2Fmain%2Fresearch%2Fdraft-Design%2520Review%2520of%2520Nova%2520%2B%2520CycleFold%2520Implementation.pdf&embedded=true"
      title="Design Review of Nova + CycleFold Implementation"
      onload="document.getElementById('pdfLoader').style.display='none';"
      allow="fullscreen">
    </iframe>
  </div>
</div>
