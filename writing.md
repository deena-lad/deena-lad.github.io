---
layout: default
title: "Writing"
description: "Case studies and technical deep-dives on Spatio-Temporal AI, Climate ML, HPC workflows, and Scientific computing."
permalink: /writing/
---

<section class="writing-page">

  <div class="page-header">
    <h1 class="page-title">Writing</h1>
    <p class="page-subtitle">Case studies on my research projects and technical deep-dives on ML systems, atmospheric modeling, HPC workflows, and research engineering.</p>
  </div>

  {% if site.posts.size > 0 %}

  <!-- ══ Build category counts ══ -->
  {% assign all_cats = "" | split: "" %}
  {% for post in site.posts %}
    {% for cat in post.categories %}
      {% assign all_cats = all_cats | push: cat %}
    {% endfor %}
  {% endfor %}
  {% assign unique_cats = all_cats | uniq | sort %}

  <div class="writing-layout">

    <!-- ── Sidebar ── -->
    <aside class="writing-sidebar">
      <p class="sidebar-label">Filter by category</p>
      <ul class="cat-filter-list">
        <li>
          <button class="cat-filter-btn active" data-cat="all">
            All
            <span class="cat-count">{{ site.posts.size }}</span>
          </button>
        </li>
        {% for cat in unique_cats %}
          {% assign count = 0 %}
          {% for post in site.posts %}
            {% if post.categories contains cat %}
              {% assign count = count | plus: 1 %}
            {% endif %}
          {% endfor %}
          {% assign cat_slug = cat | slugify %}
          <li>
            <button class="cat-filter-btn" data-cat="{{ cat_slug }}">
              {{ cat }}
              <span class="cat-count">{{ count }}</span>
            </button>
          </li>
        {% endfor %}
      </ul>
    </aside>

    <!-- ── Post list ── -->
    <div class="writing-feed">
      {% for post in site.posts %}
        {% assign post_cats = post.categories | join: " " | downcase | split: " " %}
        {% assign cat_slugs = "" %}
        {% for cat in post.categories %}
          {% assign slug = cat | slugify %}
          {% assign cat_slugs = cat_slugs | append: slug | append: " " %}
        {% endfor %}
      <article class="post-list-item" data-cats="{{ cat_slugs | strip }}">
        <div class="post-list-meta">
          <time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%b %-d, %Y" }}</time>
          {% if post.categories %}
            {% for cat in post.categories limit:2 %}
              {% assign cat_class = "cat--hpc" %}
              {% if cat == "Case Study" %}{% assign cat_class = "cat--case" %}{% endif %}
              {% if cat == "Deep Learning" or cat == "ML" %}{% assign cat_class = "cat--dl" %}{% endif %}
              {% if cat == "Satellite Imagery" %}{% assign cat_class = "cat--sat" %}{% endif %}
              {% if cat == "Quantitative Finance" or cat == "Career" %}{% assign cat_class = "cat--fin" %}{% endif %}
              <span class="post-category {{ cat_class }}">{{ cat }}</span>
            {% endfor %}
          {% endif %}
        </div>

        <h2 class="post-list-title">
          <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
        </h2>

        {% if post.excerpt %}
          <p class="post-list-excerpt">{{ post.excerpt | strip_html | truncatewords: 36 }}</p>
        {% endif %}

        {% if post.tags %}
        <div class="post-list-tags">
          {% for tag in post.tags limit:5 %}
            <span class="post-tag">{{ tag }}</span>
          {% endfor %}
        </div>
        {% endif %}

        <a href="{{ post.url | relative_url }}" class="post-list-readmore">Read article →</a>
      </article>
      {% endfor %}

      <p class="no-results" hidden>No articles in this category yet.</p>
    </div>

  </div>

  {% else %}
  <div class="blog-empty">
    <p>Case studies and articles coming soon — check back shortly.</p>
  </div>
  {% endif %}

</section>

<script>
(function () {
  const btns    = document.querySelectorAll('.cat-filter-btn');
  const posts   = document.querySelectorAll('.post-list-item');
  const noRes   = document.querySelector('.no-results');

  btns.forEach(btn => {
    btn.addEventListener('click', () => {
      const cat = btn.dataset.cat;

      /* Update active button */
      btns.forEach(b => b.classList.remove('active'));
      btn.classList.add('active');

      /* Filter posts */
      let visible = 0;
      posts.forEach(post => {
        const postCats = post.dataset.cats ? post.dataset.cats.split(' ') : [];
        const show = cat === 'all' || postCats.includes(cat);
        post.hidden = !show;
        if (show) visible++;
      });

      noRes.hidden = visible > 0;
    });
  });

  /* Activate from URL hash e.g. /writing/#case-study */
  const hash = window.location.hash.slice(1);
  if (hash) {
    const target = document.querySelector(`.cat-filter-btn[data-cat="${hash}"]`);
    if (target) target.click();
  }
})();
</script>