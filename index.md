---
layout: default
title: "Home"
description: "ML Research Engineer — Spatio-Temporal AI, Climate ML, Scientific Computing. Open to applied ML roles in quantitative finance and space-tech."
---

<!-- ══ HERO ══ -->
<section class="hero" id="sec-hero">
  <div class="hero-inner">
    <div class="hero-left">
      <p class="hero-eyebrow">ML Research Engineer</p>
      <h1 class="hero-name">Deena Lad</h1>
      <p class="hero-role">Building ML systems for climate, space &amp; scientific computing.</p>
      <div class="hero-specialty">
        <span class="spec-tag spec-tag--accent">Open to work</span>
        <span class="spec-tag">Spatio-Temporal AI</span>
        <span class="spec-tag">Climate ML</span>
        <span class="spec-tag">Scientific Computing</span>
      </div>
      <div class="hero-actions">
        <a href="/projects" class="hero-btn hero-btn--primary">View Projects</a>
        <button class="hero-btn hero-btn--resume" id="heroResumeBtn" aria-haspopup="dialog" aria-label="View and download resume">
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/></svg>
          Résumé
        </button>
        <a href="https://www.linkedin.com/in/deena-lad-307645214/" target="_blank" rel="noopener" class="hero-btn hero-btn--outline">
          <svg width="13" height="13" viewBox="0 0 24 24" fill="currentColor"><path d="M20.447 20.452h-3.554v-5.569c0-1.328-.027-3.037-1.852-3.037-1.853 0-2.136 1.445-2.136 2.939v5.667H9.351V9h3.414v1.561h.046c.477-.9 1.637-1.85 3.37-1.85 3.601 0 4.267 2.37 4.267 5.455v6.286zM5.337 7.433a2.062 2.062 0 01-2.063-2.065 2.064 2.064 0 112.063 2.065zm1.782 13.019H3.555V9h3.564v11.452zM22.225 0H1.771C.792 0 0 .774 0 1.729v20.542C0 23.227.792 24 1.771 24h20.451C23.2 24 24 23.227 24 22.271V1.729C24 .774 23.2 0 22.222 0h.003z"/></svg>
          LinkedIn
        </a>
        <a href="https://github.com/deena-lad" target="_blank" rel="noopener" class="hero-btn hero-btn--outline">
          <svg width="13" height="13" viewBox="0 0 24 24" fill="currentColor"><path d="M12 0C5.37 0 0 5.37 0 12c0 5.31 3.435 9.795 8.205 11.385.6.105.825-.255.825-.57 0-.285-.015-1.23-.015-2.235-3.015.555-3.795-.735-4.035-1.41-.135-.345-.72-1.41-1.23-1.695-.42-.225-1.02-.78-.015-.795.945-.015 1.62.87 1.845 1.23 1.08 1.815 2.805 1.305 3.495.99.105-.78.42-1.305.765-1.605-2.67-.3-5.46-1.335-5.46-5.925 0-1.305.465-2.385 1.23-3.225-.12-.3-.54-1.53.12-3.18 0 0 1.005-.315 3.3 1.23.96-.27 1.98-.405 3-.405s2.04.135 3 .405c2.295-1.56 3.3-1.23 3.3-1.23.66 1.65.24 2.88.12 3.18.765.84 1.23 1.905 1.23 3.225 0 4.605-2.805 5.625-5.475 5.925.435.375.81 1.095.81 2.22 0 1.605-.015 2.895-.015 3.3 0 .315.225.69.825.57A12.02 12.02 0 0024 12c0-6.63-5.37-12-12-12z"/></svg>
          GitHub
        </a>
      </div>
    </div>
    <div class="hero-right">
      <div class="hero-photo-wrap">
        <img
          src="/assets/images/avatar.jpg"
          alt="Deena Lad"
          class="hero-photo-img"
          onerror="this.style.display='none';this.nextElementSibling.style.display='flex';"
        />
        <div class="hero-photo-fallback">DL</div>
      </div>
    </div>
  </div>
  <!-- <div class="hero-stats">
    <div><div class="stat-val">4</div><div class="stat-label">Research projects</div></div>
    <div><div class="stat-val">3+ TB</div><div class="stat-label">Data processed</div></div>
    <div><div class="stat-val">IIT GN</div><div class="stat-label">Current affiliation</div></div>
    <div><div class="stat-val">ISRO</div><div class="stat-label">M.Tech thesis</div></div>
  </div> -->
</section>

<!-- ══ HOME BODY ══ -->
<div class="home-body">

  <!-- ── ABOUT ── -->
  <section class="home-section" id="sec-about">
    <div class="sec-hd"><span class="sec-title">About</span></div>
    <p class="about-text">
      I build <strong>deep learning systems</strong> for atmospheric science, satellite remote sensing, and medical imaging.
      Currently researching neural weather surrogates at <strong>IIT Gandhinagar's Sustainability Lab</strong> under Prof. Nipun Batra.
      M.Tech thesis at <strong>ISRO SAC</strong> on tropical cyclone intensity estimation using ConvLSTM on 1.5 TB of INSAT-3D satellite data.
      My work is directly transferable to <strong>quantitative finance</strong> and <strong>space-tech</strong>.
    </p>
  </section>

  <!-- ── PROJECTS ── -->
  <section class="home-section" id="sec-projects">
    <div class="sec-hd">
      <span class="sec-title">Projects</span>
      <a href="/projects" class="sec-link">View all →</a>
    </div>
    <div class="proj-grid">
      {% for project in site.data.projects limit:4 %}
      {% unless project.coming_soon %}
      <div class="proj-card" data-project-link="/projects#{{ project.id }}" tabindex="0" role="button">
        <div class="proj-card-top">
          <span class="proj-tag tag--{{ project.tag_color }}">{{ project.tag }}</span>
          <span class="proj-inst">
            {% for badge in project.badges limit:1 %}{{ badge.label }}{% endfor %}
          </span>
        </div>
        <div class="proj-title">{{ project.title }}</div>
        <div class="proj-desc">{{ project.short_desc }}</div>
        <div class="proj-chips">
          {% for tag in project.tech_tags limit:3 %}
            <code class="tech-chip">{{ tag }}</code>
          {% endfor %}
          {% if project.tech_tags.size > 3 %}
            <code class="tech-chip tech-chip--more">+{{ project.tech_tags.size | minus: 3 }}</code>
          {% endif %}
          {% if project.github or project.demo %}
            <span class="cs-badge">Case Study</span>
          {% endif %}
        </div>
      </div>
      {% endunless %}
      {% endfor %}
    </div>
  </section>

  <!-- ── SKILLS ── -->
  <section class="home-section" id="sec-skills">
    <div class="sec-hd">
      <span class="sec-title">Skills</span>
      <a href="/skills" class="sec-link">Full breakdown →</a>
    </div>
    <div class="skills-home-grid">
      <div class="skill-home-card">
        <div class="skill-home-title"><div class="skill-dot"></div>Deep Learning</div>
        <div class="skill-home-tags">
          <span class="skill-home-tag">PyTorch</span>
          <span class="skill-home-tag">U-Net</span>
          <span class="skill-home-tag">ConvLSTM</span>
          <span class="skill-home-tag">Transformers</span>
          <span class="skill-home-tag">YOLO</span>
        </div>
      </div>
      <div class="skill-home-card">
        <div class="skill-home-title"><div class="skill-dot"></div>Geospatial &amp; Scientific</div>
        <div class="skill-home-tags">
          <span class="skill-home-tag">xarray</span>
          <span class="skill-home-tag">NetCDF</span>
          <span class="skill-home-tag">GeoPandas</span>
          <span class="skill-home-tag">ERA5</span>
          <span class="skill-home-tag">INSAT-3D</span>
        </div>
      </div>
      <div class="skill-home-card">
        <div class="skill-home-title"><div class="skill-dot"></div>MLOps &amp; Infra</div>
        <div class="skill-home-tags">
          <span class="skill-home-tag">SLURM</span>
          <span class="skill-home-tag">Docker</span>
          <span class="skill-home-tag">W&amp;B</span>
          <span class="skill-home-tag">GitHub Actions</span>
        </div>
      </div>
      <div class="skill-home-card">
        <div class="skill-home-title"><div class="skill-dot"></div>Generative AI &amp; NLP</div>
        <div class="skill-home-tags">
          <span class="skill-home-tag">LangChain</span>
          <span class="skill-home-tag">RAG</span>
          <span class="skill-home-tag">HuggingFace</span>
          <span class="skill-home-tag">BERT</span>
        </div>
      </div>
      <div class="skill-home-card">
        <div class="skill-home-title"><div class="skill-dot"></div>Languages &amp; Data</div>
        <div class="skill-home-tags">
          <span class="skill-home-tag">Python</span>
          <span class="skill-home-tag">SQL</span>
          <span class="skill-home-tag">NumPy</span>
          <span class="skill-home-tag">Pandas</span>
        </div>
      </div>
      <div class="skill-home-card">
        <div class="skill-home-title"><div class="skill-dot"></div>Cloud &amp; Viz</div>
        <div class="skill-home-tags">
          <span class="skill-home-tag">GCP / Vertex AI</span>
          <span class="skill-home-tag">BigQuery</span>
          <span class="skill-home-tag">Plotly</span>
        </div>
      </div>
    </div>
  </section>

  <!-- ── WRITING ── -->
  <section class="home-section" id="sec-writing">
    <div class="sec-hd">
      <span class="sec-title">Writing</span>
      <a href="/writing" class="sec-link">All posts →</a>
    </div>
    <div class="writing-list">
      {% assign writing_posts = site.posts | limit: 4 %}
      {% if writing_posts.size > 0 %}
        {% for post in writing_posts %}
        <div class="writing-row">
          {% if post.categories contains "Case Study" %}
            <span class="writing-type wt-case">Case Study</span>
          {% else %}
            <span class="writing-type wt-blog">Blog</span>
          {% endif %}
          <div>
            <div class="writing-title"><a href="{{ post.url | relative_url }}" style="color:inherit;text-decoration:none;">{{ post.title }}</a></div>
            <div class="writing-meta">{{ post.date | date: "%b %-d, %Y" }} · {{ post.categories | join: " · " }}</div>
          </div>
          <a href="{{ post.url | relative_url }}" class="writing-arrow" aria-label="Read {{ post.title }}">→</a>
        </div>
        {% endfor %}
      {% else %}
      <div class="writing-row">
        <span class="writing-type wt-case">Case Study</span>
        <div>
          <div class="writing-title">How I built a neural surrogate 10× faster than WRF</div>
          <div class="writing-meta">Neural Weather Forecasting · IIT Gandhinagar · Coming soon</div>
        </div>
        <span class="writing-arrow">→</span>
      </div>
      <div class="writing-row">
        <span class="writing-type wt-case">Case Study</span>
        <div>
          <div class="writing-title">Estimating cyclone intensity from INSAT-3D with ConvLSTM</div>
          <div class="writing-meta">Tropical Cyclone Analysis · ISRO SAC · Coming soon</div>
        </div>
        <span class="writing-arrow">→</span>
      </div>
      {% endif %}
    </div>
  </section>

  <!-- ── EXPERIENCE ── -->
  <section class="home-section" id="sec-experience">
    <div class="sec-hd"><span class="sec-title">Experience</span></div>
    <div class="exp-list">

      <div class="exp-item">
        <div class="exp-line">
          <div class="exp-dot"></div>
          <div class="exp-connector"></div>
        </div>
        <div class="exp-card">
          <div class="exp-header">
            <span class="exp-role">Junior Research Fellow</span>
            <span class="exp-date">Aug 2025 — Present</span>
          </div>
          <p class="exp-org">Sustainability Lab, IIT Gandhinagar · PI: Prof. Nipun Batra</p>
          <ul class="exp-bullets">
            <li>End-to-end ML pipeline on 500 GB WRF + 2+ TB ERA5 data — scalable NetCDF loading, distributed preprocessing, GPU training on HPC (SLURM).</li>
            <li>Spatio-temporal surrogate models (U-Net + FiLM, Transformers) for 3–24 h forecasting; RMSE ≈ 2.4 °C — <strong>10× faster than WRF baselines</strong>.</li>
            <li>Co-authoring paper; mentoring 3 junior students.</li>
          </ul>
          <span class="exp-badge exp-badge--ongoing">⬤ Ongoing Research</span>
        </div>
      </div>

      <div class="exp-item">
        <div class="exp-line">
          <div class="exp-dot exp-dot--violet"></div>
          <div class="exp-connector"></div>
        </div>
        <div class="exp-card">
          <div class="exp-header">
            <span class="exp-role">Research Intern — Deep Learning for Satellite Imagery</span>
            <span class="exp-date">Jul 2024 — Apr 2025</span>
          </div>
          <p class="exp-org">Space Applications Centre, ISRO · Ahmedabad</p>
          <ul class="exp-bullets">
            <li>CNN + ConvLSTM models for tropical cyclone intensity estimation and next-frame forecasting on 1.5+ TB INSAT-3D HDF5 data (2013–2023).</li>
            <li>End-to-end preprocessing pipeline using IMD track data and Advanced Dvorak Technique; curated 10K annotated images.</li>
            <li>Delivered operational storm-evolution models — M.Tech thesis, graded outstanding.</li>
          </ul>
          <span class="exp-badge exp-badge--thesis">M.Tech Thesis Project</span>
        </div>
      </div>

      <div class="exp-item">
        <div class="exp-line">
          <div class="exp-dot exp-dot--teal"></div>
        </div>
        <div class="exp-card">
          <div class="exp-header">
            <span class="exp-role">ML / Data Engineering Intern</span>
            <span class="exp-date">May 2024 — Jul 2024</span>
          </div>
          <p class="exp-org">Jio Platforms Limited, Reliance · Remote</p>
          <ul class="exp-bullets">
            <li>Built scraping and ETL pipelines for multi-modal sports video datasets for athlete personality analysis.</li>
            <li>Applied prompt engineering via Groq API to generate structured LLM annotations at scale.</li>
          </ul>
        </div>
      </div>

    </div>
  </section>

  <!-- ── EDUCATION ── -->
  <section class="home-section" id="sec-education">
    <div class="sec-hd"><span class="sec-title">Education</span></div>
    <div class="edu-list">

      <div class="edu-card">
        <div>
          <p class="edu-degree">M.Tech in Data Science</p>
          <p class="edu-school">Pandit Deendayal Energy University, Gandhinagar</p>
          <p class="edu-detail">GPA: 9.61 / 10</p>
          <p class="edu-detail">Thesis: <a href="/projects#cyclone-analysis">Tropical Cyclone Intensity Estimation using Deep Learning</a> - ISRO SAC</p>
          <span class="edu-medal">🥇 Gold Medalist - Top Graduate of Program</span>
        </div>
        <span class="edu-date">Jul 2023 — May 2025</span>
      </div>

      <div class="edu-card">
        <div>
          <p class="edu-degree">B.E. in Computer Science &amp; Engineering</p>
          <p class="edu-school">Babaria Institute of Technology (GTU), Vadodara</p>
          <p class="edu-detail">GPA: 9.02 / 10</p>
        </div>
        <span class="edu-date">Jul 2019 — May 2023</span>
      </div>

    </div>
  </section>

  <!-- ── COMMUNITY ── -->
  <section class="home-section" id="sec-community">
    <div class="sec-hd"><span class="sec-title">Leadership &amp; Community</span></div>
    <div class="community-list">

      <div class="comm-card">
        <div class="comm-icon">🔬</div>
        <div>
          <p class="comm-title">IEEE Computer Society, PDEU — Vice Chair → Chair → Advisor</p>
          <p class="comm-sub">2023–2025 · Led technical workshops, hackathons, and community initiatives for 200+ student members.</p>
        </div>
      </div>

      <div class="comm-card">
        <div class="comm-icon">🎨</div>
        <div>
          <p class="comm-title">Google Developer Student Club (GDSC), BIT — Design Team</p>
          <p class="comm-sub">2021–2022 · Created visual assets and managed design communications for developer events.</p>
        </div>
      </div>

      <div class="comm-card">
        <div class="comm-icon">🌱</div>
        <div>
          <p class="comm-title">National Service Scheme (NSS), GTU — Volunteer</p>
          <p class="comm-sub">2020–2023 · Community service, rural outreach, and social welfare programmes.</p>
        </div>
      </div>

      <div class="comm-card">
        <div class="comm-icon">📜</div>
        <div>
          <p class="comm-title">Certifications</p>
          <p class="comm-sub">IBM Generative AI Engineering with LLMs (7-course) · Google Cloud Skills Boost (30-Day) · Looker Studio Advanced</p>
        </div>
      </div>

    </div>
  </section>

</div><!-- end .home-body -->

<!-- ══ RESUME MODAL ══ -->
<div id="resumeModal" class="resume-modal-overlay" role="dialog" aria-modal="true" aria-label="Resume viewer" hidden>
  <div class="resume-modal">
    <div class="resume-modal-header">
      <span class="resume-modal-title">Deena Lad — Résumé</span>
      <div class="resume-modal-actions">
        <a href="/assets/resume/Deena_Lad_Resume.pdf" download class="resume-dl-btn">
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15v4a2 2 0 01-2 2H5a2 2 0 01-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/></svg>
          Download PDF
        </a>
        <button class="resume-modal-close" id="resumeModalClose" aria-label="Close">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
        </button>
      </div>
    </div>
    <iframe src="/assets/files/resume.pdf" class="resume-embed" title="Deena Lad Resume PDF"></iframe>
  </div>
</div>

<!-- ══ JS ══ -->
<script>
(function(){
  /* Resume modal */
  const modal    = document.getElementById('resumeModal');
  const closeBtn = document.getElementById('resumeModalClose');
  const heroBtn  = document.getElementById('heroResumeBtn');
  const navBtn   = document.getElementById('navResumeBtn');

  function openResume()  { modal.hidden = false; document.body.classList.add('modal-open'); setTimeout(()=>closeBtn.focus(),50); }
  function closeResume() { modal.hidden = true;  document.body.classList.remove('modal-open'); }

  if(heroBtn)  heroBtn.addEventListener('click', openResume);
  if(navBtn)   navBtn.addEventListener('click', openResume);
  if(closeBtn) closeBtn.addEventListener('click', closeResume);
  modal.addEventListener('click', e => { if(e.target===modal) closeResume(); });
  document.addEventListener('keydown', e => { if(e.key==='Escape' && !modal.hidden) closeResume(); });

  /* Project cards on homepage → navigate to projects page */
  document.querySelectorAll('.proj-card[data-project-link]').forEach(card => {
    card.addEventListener('click', () => { window.location.href = card.dataset.projectLink; });
    card.addEventListener('keydown', e => { if(e.key==='Enter'||e.key===' '){ e.preventDefault(); window.location.href = card.dataset.projectLink; }});
  });
})();
</script>