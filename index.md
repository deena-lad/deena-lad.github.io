---
layout: default
title: "Home"
description: "M.Tech Gold Medalist & ML Researcher specializing in Spatio-Temporal AI, Climate ML, and Scientific ML."
---
<div class="site-notice">
  🚧 Portfolio Update in Progress — I am currently redesigning and expanding this portfolio. New projects, blogs and research highlights are being added. The updated version will be completed by <strong>Monday, 8 June 2026</strong>.
</div>

<section class="hero">
  <div class="hero-inner">

    <div class="hero-avatar">
      <div class="avatar-ring">
        <img src="/assets/images/avatar.jpg" alt="Deena Lad" class="avatar-img" onerror="this.style.display='none'; this.nextElementSibling.style.display='flex';" />
        <div class="avatar-fallback">DL</div>
      </div>
    </div>

    <div class="hero-content">
      <p class="hero-eyebrow">M.Tech Gold Medalist &nbsp;·&nbsp; ML Research Engineer</p>
      <h1 class="hero-name">Deena Lad</h1>
      <p class="hero-tagline">
        Building AI systems that reason over space, time, and the physical world.
      </p>

      <div class="hero-domains">
        <span class="domain-chip">Spatio-Temporal AI</span>
        <span class="domain-chip">Climate ML</span>
        <span class="domain-chip">Scientific ML</span>
        <span class="domain-chip">Computer Vision</span>
        <span class="domain-chip">HPC & Geospatial</span>
      </div>

      <p class="hero-bio">
        I design and train deep learning pipelines that process terabytes of geospatial and satellite data, from atmospheric reanalysis fields to INSAT-3D imagery. My research sits at the intersection of physics-informed modeling, sequence learning, and high-performance computing. Have worked on neural weather forecasting at IITGN and tropical cyclone analysis at ISRO.
      </p>

      <div class="hero-links">
        <a href="deenalad06@gmail.com" class="hero-btn hero-btn--primary">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/><polyline points="22,6 12,13 2,6"/></svg>
          Email
        </a>
        <a href="https://www.linkedin.com/in/deena-lad-307645214/" target="_blank" rel="noopener" class="hero-btn hero-btn--outline">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M20.447 20.452h-3.554v-5.569c0-1.328-.027-3.037-1.852-3.037-1.853 0-2.136 1.445-2.136 2.939v5.667H9.351V9h3.414v1.561h.046c.477-.9 1.637-1.85 3.37-1.85 3.601 0 4.267 2.37 4.267 5.455v6.286zM5.337 7.433a2.062 2.062 0 01-2.063-2.065 2.064 2.064 0 112.063 2.065zm1.782 13.019H3.555V9h3.564v11.452zM22.225 0H1.771C.792 0 0 .774 0 1.729v20.542C0 23.227.792 24 1.771 24h20.451C23.2 24 24 23.227 24 22.271V1.729C24 .774 23.2 0 22.222 0h.003z"/></svg>
          LinkedIn
        </a>
        <a href="https://github.com/deena-lad" target="_blank" rel="noopener" class="hero-btn hero-btn--outline">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M12 0C5.37 0 0 5.37 0 12c0 5.31 3.435 9.795 8.205 11.385.6.105.825-.255.825-.57 0-.285-.015-1.23-.015-2.235-3.015.555-3.795-.735-4.035-1.41-.135-.345-.72-1.41-1.23-1.695-.42-.225-1.02-.78-.015-.795.945-.015 1.62.87 1.845 1.23 1.08 1.815 2.805 1.305 3.495.99.105-.78.42-1.305.765-1.605-2.67-.3-5.46-1.335-5.46-5.925 0-1.305.465-2.385 1.23-3.225-.12-.3-.54-1.53.12-3.18 0 0 1.005-.315 3.3 1.23.96-.27 1.98-.405 3-.405s2.04.135 3 .405c2.295-1.56 3.3-1.23 3.3-1.23.66 1.65.24 2.88.12 3.18.765.84 1.23 1.905 1.23 3.225 0 4.605-2.805 5.625-5.475 5.925.435.375.81 1.095.81 2.22 0 1.605-.015 2.895-.015 3.3 0 .315.225.69.825.57A12.02 12.02 0 0024 12c0-6.63-5.37-12-12-12z"/></svg>
          GitHub
        </a>
        <a href="/projects" class="hero-btn hero-btn--outline">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="2" y="3" width="20" height="14" rx="2"/><line x1="8" y1="21" x2="16" y2="21"/><line x1="12" y1="17" x2="12" y2="21"/></svg>
          Portfolio
        </a>
      </div>
    </div>

  </div>
</section>

<!-- ══ HIGHLIGHTS STRIP ══ -->
<section class="highlights">
  <div class="highlights-grid">
    <div class="highlight-card">
      <span class="highlight-number">3+ TB</span>
      <span class="highlight-label">Atmospheric data processed</span>
    </div>
    <div class="highlight-card">
      <span class="highlight-number">10×</span>
      <span class="highlight-label">Speedup over physics baselines</span>
    </div>
    <div class="highlight-card">
      <span class="highlight-number">10K+</span>
      <span class="highlight-label">Annotated satellite images</span>
    </div>
  </div>
</section>

<!-- ══ RECENT PROJECTS TEASER ══ -->
<section class="home-section">
  <div class="section-header">
    <h2 class="section-title">Selected Research</h2>
    <a href="/projects" class="section-link">View all projects →</a>
  </div>
  <div class="teaser-grid">
    {% for project in site.data.projects limit:3 %}
    {% unless project.coming_soon %}
    <div class="teaser-card">
      <span class="teaser-tag tag--{{ project.tag_color }}">{{ project.tag }}</span>
      <h3 class="teaser-title">{{ project.title }}</h3>
      <p class="teaser-desc">{{ project.short_desc }}</p>
    </div>
    {% endunless %}
    {% endfor %}
  </div>
</section>
