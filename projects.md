---
layout: default
title: "Projects"
description: "ML research projects spanning atmospheric AI, satellite remote sensing, and medical imaging which are production-grade pipelines on multi-terabyte datasets."
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

    <button class="modal-close" id="modalClose" aria-label="Close modal">
      <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
    </button>

    <!-- Image gallery -->
    <div class="modal-gallery" id="modalGallery">
      <div class="gallery-main" id="galleryMain">
        <!-- main image injected by JS -->
      </div>
      <div class="gallery-caption" id="galleryCaption"></div>
      <div class="gallery-thumbs" id="galleryThumbs">
        <!-- thumbnails injected by JS -->
      </div>
      <!-- Nav arrows -->
      <button class="gallery-arrow gallery-arrow--prev" id="galleryPrev" aria-label="Previous image">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="15 18 9 12 15 6"/></svg>
      </button>
      <button class="gallery-arrow gallery-arrow--next" id="galleryNext" aria-label="Next image">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="9 18 15 12 9 6"/></svg>
      </button>
    </div>

    <div class="modal-content">
      <div class="modal-context" id="modalContext"></div>
      <span class="modal-tag" id="modalTag"></span>
      <h2 class="modal-title" id="modalTitle"></h2>
      <p class="modal-institution" id="modalInstitution"></p>
      <p class="modal-long-desc" id="modalDesc"></p>
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
      gallery:     {{ project.gallery | jsonify }}
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