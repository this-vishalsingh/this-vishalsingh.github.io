---
layout: default
title: Blogs
active: blog
permalink: /blog/
---

<div class="search-section" style="margin-bottom: 2rem;">
  <div class="search-bar-wrapper">
    <input type="text" id="blogSearch" placeholder="Filter research & checklists..." onkeyup="filterBlogs()">
    <i class="search-icon">🔍</i>
  </div>
</div>

### CRYPTOGRAPHY/SEC Quarterly

<ul id="blogList" style="list-style: none; padding: 0; margin: 0;">
  <li class="blog-item"><a href="/blog/fhe-audit-checklist/">FHE Protocol Security Audit Checklist</a></li>
  <li class="blog-item"><a href="/blog/nova-cyclefold/">Nova and CycleFold</a></li>
  <li class="blog-item"><a href="/blog/cairo-security-checklist/">Cairo Security Vulnerabilities Checklist</a></li>
  <li class="blog-item"><a href="/blog/lean-multisig-bugs/">Breaking LeanMultisig zkVM Shamir (Part 1)</a></li>
  <li class="blog-item"><a href="/blog/audit-process/">My Security Audit Process</a></li>
  <li class="blog-item"><a href="/blog/zksnarks-security">ZK-SNARKs Security - Vulnerabilities and Root Causes</a></li>
</ul>

<script>
function filterBlogs() {
  const input = document.getElementById('blogSearch');
  const filter = input.value.toLowerCase();
  const list = document.getElementById('blogList');
  const items = list.getElementsByTagName('li');
  
  for (let i = 0; i < items.length; i++) {
    const text = items[i].textContent || items[i].innerText;
    if (text.toLowerCase().indexOf(filter) > -1) {
      items[i].style.display = "";
      items[i].style.opacity = "1";
    } else {
      items[i].style.display = "none";
      items[i].style.opacity = "0";
    }
  }
}
</script>
