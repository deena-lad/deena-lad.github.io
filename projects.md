---
layout: default
title: "Projects"
description: "ML research projects in Spatio-Temporal AI, Climate ML, Medical AI, and Scientific computing."
---

<section class="projects-section">

  <div class="page-header">
    <h1 class="page-title">Projects</h1>
    <p class="page-subtitle">Research spanning atmospheric science, satellite remote sensing, and medical imaging — built on multi-terabyte datasets with production-grade ML pipelines.</p>
  </div>

  <!-- ══ PROJECT GRID ══ -->
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
    >
      <!-- Card image / placeholder -->
      <div class="card-img-wrap">
        {% if project.image and project.image != "" %}
          <img src="{{ project.image | relative_url }}" alt="{{ project.title }}" class="card-img" loading="lazy" />
        {% else %}
          <div class="card-img-placeholder">{{ project.image_placeholder }}</div>
        {% endif %}
        {% if project.coming_soon %}
          <div class="coming-soon-badge">Coming Soon</div>
        {% endif %}
      </div>

      <!-- Card body -->
      <div class="card-body">
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

<!-- ══════════════════════════════════════
     MODAL OVERLAY
══════════════════════════════════════ -->
<div id="projectModal" class="modal-overlay" role="dialog" aria-modal="true" aria-labelledby="modalTitle" hidden>
  <div class="modal-container">

    <button class="modal-close" id="modalClose" aria-label="Close modal">
      <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
        <line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/>
      </svg>
    </button>

    <div class="modal-img-wrap" id="modalImgWrap">
      <!-- Filled by JS -->
    </div>

    <div class="modal-content">
      <span class="modal-tag" id="modalTag"></span>
      <h2 class="modal-title" id="modalTitle"></h2>
      <p class="modal-long-desc" id="modalDesc"></p>

      <div class="modal-tech-section">
        <p class="modal-tech-label">Tech Stack</p>
        <div class="modal-tech-tags" id="modalTechTags">
          <!-- Filled by JS -->
        </div>
      </div>
    </div>

  </div>
</div>

<!-- ══ Data store + Modal JS ══ -->
<script>
(function () {
  /* ── Project data from Jekyll ── */
  const projects = {
    {% for project in site.data.projects %}{% unless project.coming_soon %}"{{ project.id }}": {
      title: {{ project.title | jsonify }},
      tag:   {{ project.tag   | jsonify }},
      tagColor: {{ project.tag_color | jsonify }},
      image: {{ project.image | default: "" | jsonify }},
      placeholder: {{ project.image_placeholder | jsonify }},
      desc:  {{ project.long_desc  | jsonify }},
      tech:  {{ project.tech_tags  | jsonify }}
    },{% endunless %}{% endfor %}
  };

  /* ── DOM refs ── */
  const overlay   = document.getElementById('projectModal');
  const closeBtn  = document.getElementById('modalClose');
  const modalTag  = document.getElementById('modalTag');
  const modalTitle= document.getElementById('modalTitle');
  const modalDesc = document.getElementById('modalDesc');
  const modalTech = document.getElementById('modalTechTags');
  const modalImg  = document.getElementById('modalImgWrap');
  let lastFocused = null;

  /* ── Open ── */
  function openModal(id) {
    const p = projects[id];
    if (!p) return;

    lastFocused = document.activeElement;

    /* Populate */
    modalTag.textContent  = p.tag;
    modalTag.className    = `modal-tag tag--${p.tagColor}`;
    modalTitle.textContent= p.title;
    modalDesc.textContent = p.desc;

    /* Image */
    modalImg.innerHTML = p.image
      ? `<img src="${p.image}" alt="${p.title}" class="modal-img" />`
      : `<div class="modal-img-placeholder">${p.placeholder}</div>`;

    /* Tech tags */
    modalTech.innerHTML = p.tech
      .map(t => `<code class="tech-chip">${t}</code>`)
      .join('');

    /* Show */
    overlay.hidden = false;
    document.body.classList.add('modal-open');

    /* Trap focus to close button initially */
    setTimeout(() => closeBtn.focus(), 50);
  }

  /* ── Close ── */
  function closeModal() {
    overlay.hidden = true;
    document.body.classList.remove('modal-open');
    if (lastFocused) lastFocused.focus();
  }

  /* ── Event listeners ── */
  document.querySelectorAll('.project-card[data-project]').forEach(card => {
    card.addEventListener('click', () => openModal(card.dataset.project));
    card.addEventListener('keydown', e => {
      if (e.key === 'Enter' || e.key === ' ') {
        e.preventDefault();
        openModal(card.dataset.project);
      }
    });
  });

  closeBtn.addEventListener('click', closeModal);

  /* Click outside modal container */
  overlay.addEventListener('click', e => {
    if (e.target === overlay) closeModal();
  });

  /* Escape key */
  document.addEventListener('keydown', e => {
    if (e.key === 'Escape' && !overlay.hidden) closeModal();
  });
})();
</script>
