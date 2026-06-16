---
layout: default
title: "Projects"
description: "ML research projects spanning atmospheric AI, satellite remote sensing, and medical imaging — production-grade pipelines on multi-terabyte datasets."
---

<section class="projects-section">
  <div class="page-header">
    <h1 class="page-title">Projects</h1>
    <p class="page-subtitle">Research spanning atmospheric science, satellite remote sensing, and medical imaging — production-grade ML pipelines built on multi-terabyte scientific datasets, with direct applicability to quantitative finance and space-tech.</p>
  </div>

  <div class="projects-grid">
    {% for project in site.data.projects %}
    <article
      class="project-card {% if project.coming_soon %}project-card--soon{% endif %}"
      {% unless project.coming_soon %}
      data-project="{{ project.id }}"
      tabindex="0"
      role="button"
      aria-haspopup="dialog"
      aria-label="View details for {{ project.title }}"
      {% endunless %}
      {% if project.id == 'cyclone-analysis' %}id="cyclone-analysis"{% endif %}
    >
      <div class="card-img-wrap">
        {% if project.gallery and project.gallery.size > 0 %}
          <img src="{{ project.gallery[0].src | relative_url }}" alt="{{ project.title }}" class="card-img" loading="lazy" />
        {% else %}
          <div class="card-img-placeholder">{{ project.image_placeholder }}</div>
        {% endif %}
        {% if project.coming_soon %}<div class="coming-soon-badge">Coming Soon</div>{% endif %}
      </div>
      <div class="card-body">
        {% if project.badges and project.badges.size > 0 %}
        <div class="card-context">
          {% for badge in project.badges %}
            <span class="card-context-badge badge--{{ badge.style }}">{{ badge.label }}</span>
          {% endfor %}
        </div>
        {% endif %}
        <span class="card-tag tag--{{ project.tag_color }}">{{ project.tag }}</span>
        <h2 class="card-title">{{ project.title }}</h2>
        <p class="card-desc">{{ project.short_desc }}</p>
        {% unless project.coming_soon %}
        <div class="card-tech-preview">
          {% for tag in project.tech_tags limit:3 %}
            <code class="tech-chip">{{ tag }}</code>
          {% endfor %}
          {% if project.tech_tags.size > 3 %}
            <code class="tech-chip tech-chip--more">+{{ project.tech_tags.size | minus: 3 }}</code>
          {% endif %}
        </div>
        <button class="card-cta" tabindex="-1" aria-hidden="true">View Case Study →</button>
        {% endunless %}
      </div>
    </article>
    {% endfor %}
  </div>
</section>

<!-- ══ MODAL ══ -->
<div id="projectModal" class="modal-overlay" role="dialog" aria-modal="true" aria-labelledby="modalTitle" hidden>
  <div class="modal-container modal-wide">

    <!-- Image gallery -->
    <div class="modal-gallery" id="modalGallery">
      <div class="gallery-main" id="galleryMain"></div>
      <div class="gallery-caption" id="galleryCaption"></div>
      <div class="gallery-thumbs" id="galleryThumbs"></div>
      <button class="gallery-arrow gallery-arrow--prev" id="galleryPrev" aria-label="Previous image">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="15 18 9 12 15 6"/></svg>
      </button>
      <button class="gallery-arrow gallery-arrow--next" id="galleryNext" aria-label="Next image">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="9 18 15 12 9 6"/></svg>
      </button>
    </div>

    <div class="modal-content">
      <!-- Close button lives inside content so position:absolute works -->
      <button class="modal-close" id="modalClose" aria-label="Close modal">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
      </button>
      <div class="modal-context" id="modalContext"></div>
      <span class="modal-tag" id="modalTag"></span>
      <h2 class="modal-title" id="modalTitle"></h2>
      <p class="modal-institution" id="modalInstitution"></p>
      <p class="modal-long-desc" id="modalDesc"></p>
      <div class="modal-links" id="modalLinks"></div>
      <p class="modal-tech-label">Tech Stack</p>
      <div class="modal-tech-tags" id="modalTechTags"></div>
    </div>

  </div>
</div>

<!-- ══ DATA + JS ══ -->
<script>
(function(){
  const projects = {
    {% for project in site.data.projects %}{% unless project.coming_soon %}"{{ project.id }}": {
      title:       {{ project.title | jsonify }},
      tag:         {{ project.tag | jsonify }},
      tagColor:    {{ project.tag_color | jsonify }},
      desc:        {{ project.long_desc | jsonify }},
      tech:        {{ project.tech_tags | jsonify }},
      institution: {{ project.institution | default: "" | jsonify }},
      badges:      {{ project.badges | jsonify }},
      gallery:     {{ project.gallery | jsonify }},
      github:      {{ project.github | default: "" | jsonify }},
      demo:        {{ project.demo | default: "" | jsonify }}
    },{% endunless %}{% endfor %}
  };

  const overlay    = document.getElementById('projectModal');
  const closeBtn   = document.getElementById('modalClose');
  const galleryMain   = document.getElementById('galleryMain');
  const galleryThumbs = document.getElementById('galleryThumbs');
  const galleryCaption= document.getElementById('galleryCaption');
  const galleryPrev   = document.getElementById('galleryPrev');
  const galleryNext   = document.getElementById('galleryNext');

  let currentGallery = [];
  let currentIndex   = 0;
  let lastFocused    = null;

  function renderGallery(idx) {
    currentIndex = idx;
    const item = currentGallery[idx];

    // Main image
    galleryMain.innerHTML = `<img src="${item.src}" alt="${item.caption}" class="gallery-main-img" onerror="this.style.opacity='.3'" />`;

    // Caption
    galleryCaption.textContent = item.caption;

    // Thumbnails
    galleryThumbs.innerHTML = currentGallery.map((g, i) => `
      <button class="gallery-thumb ${i === idx ? 'active' : ''}" onclick="selectImage(${i})" aria-label="View image ${i+1}">
        <img src="${g.src}" alt="" onerror="this.style.opacity='.2'" />
        ${g.src.includes('placeholder') ? '<span class="thumb-placeholder">📷</span>' : ''}
      </button>
    `).join('');

    // Arrow visibility
    galleryPrev.style.opacity = idx === 0 ? '.3' : '1';
    galleryPrev.disabled = idx === 0;
    galleryNext.style.opacity = idx === currentGallery.length - 1 ? '.3' : '1';
    galleryNext.disabled = idx === currentGallery.length - 1;
  }

  window.selectImage = function(idx) { renderGallery(idx); };

  galleryPrev.addEventListener('click', () => { if(currentIndex > 0) renderGallery(currentIndex - 1); });
  galleryNext.addEventListener('click', () => { if(currentIndex < currentGallery.length - 1) renderGallery(currentIndex + 1); });

  function openModal(id) {
    const p = projects[id]; if(!p) return;
    lastFocused = document.activeElement;

    // Gallery
    currentGallery = (p.gallery || []).map(g => ({
      src: g.src || '',
      caption: g.caption || ''
    }));
    if(currentGallery.length === 0) currentGallery = [{ src: '', caption: p.title }];
    renderGallery(0);

    // Text
    document.getElementById('modalTag').textContent = p.tag;
    document.getElementById('modalTag').className   = `modal-tag tag--${p.tagColor}`;
    document.getElementById('modalTitle').textContent = p.title;
    document.getElementById('modalDesc').textContent  = p.desc;
    const inst = document.getElementById('modalInstitution');
    inst.textContent  = p.institution || '';
    inst.style.display = p.institution ? '' : 'none';
    document.getElementById('modalContext').innerHTML = (p.badges||[]).map(b =>
      `<span class="card-context-badge badge--${b.style}">${b.label}</span>`).join('');
    document.getElementById('modalTechTags').innerHTML = (p.tech||[]).map(t =>
      `<code class="tech-chip">${t}</code>`).join('');

    // GitHub / Demo links
    const linksEl = document.getElementById('modalLinks');
    const linkHTML = [];
    if(p.github) linkHTML.push(`<a href="${p.github}" target="_blank" rel="noopener" class="modal-link-btn modal-link-btn--github">
      <svg width="14" height="14" viewBox="0 0 24 24" fill="currentColor"><path d="M12 0C5.37 0 0 5.37 0 12c0 5.31 3.435 9.795 8.205 11.385.6.105.825-.255.825-.57 0-.285-.015-1.23-.015-2.235-3.015.555-3.795-.735-4.035-1.41-.135-.345-.72-1.41-1.23-1.695-.42-.225-1.02-.78-.015-.795.945-.015 1.62.87 1.845 1.23 1.08 1.815 2.805 1.305 3.495.99.105-.78.42-1.305.765-1.605-2.67-.3-5.46-1.335-5.46-5.925 0-1.305.465-2.385 1.23-3.225-.12-.3-.54-1.53.12-3.18 0 0 1.005-.315 3.3 1.23.96-.27 1.98-.405 3-.405s2.04.135 3 .405c2.295-1.56 3.3-1.23 3.3-1.23.66 1.65.24 2.88.12 3.18.765.84 1.23 1.905 1.23 3.225 0 4.605-2.805 5.625-5.475 5.925.435.375.81 1.095.81 2.22 0 1.605-.015 2.895-.015 3.3 0 .315.225.69.825.57A12.02 12.02 0 0024 12c0-6.63-5.37-12-12-12z"/></svg>
      View on GitHub</a>`);
    if(p.demo) linkHTML.push(`<a href="${p.demo}" target="_blank" rel="noopener" class="modal-link-btn modal-link-btn--demo">
      <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M18 13v6a2 2 0 01-2 2H5a2 2 0 01-2-2V8a2 2 0 012-2h6"/><polyline points="15 3 21 3 21 9"/><line x1="10" y1="14" x2="21" y2="3"/></svg>
      Live Demo</a>`);
    linksEl.innerHTML = linkHTML.join('');
    linksEl.style.display = linkHTML.length ? '' : 'none';

    overlay.hidden = false;
    document.body.classList.add('modal-open');
    setTimeout(() => closeBtn.focus(), 50);
  }

  function closeModal() {
    overlay.hidden = true;
    document.body.classList.remove('modal-open');
    if(lastFocused) lastFocused.focus();
  }

  document.querySelectorAll('.project-card[data-project]').forEach(card => {
    card.addEventListener('click',    () => openModal(card.dataset.project));
    card.addEventListener('keydown',  e  => { if(e.key==='Enter'||e.key===' '){ e.preventDefault(); openModal(card.dataset.project); }});
  });
  closeBtn.addEventListener('click', closeModal);
  overlay.addEventListener('click',  e => { if(e.target===overlay) closeModal(); });
  document.addEventListener('keydown', e => {
    if(e.key==='Escape' && !overlay.hidden) closeModal();
    if(e.key==='ArrowLeft'  && !overlay.hidden) galleryPrev.click();
    if(e.key==='ArrowRight' && !overlay.hidden) galleryNext.click();
  });

  if(window.location.hash === '#cyclone-analysis') {
    setTimeout(() => openModal('cyclone-analysis'), 300);
  }
})();
</script>