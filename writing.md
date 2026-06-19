---
layout: default
title: "Writing"
description: "Case studies and technical deep-dives on Spatio-Temporal AI, Climate ML, HPC workflows, and Scientific computing."
permalink: /writing/
---

<section class="blog-section">

  <div class="page-header">
    <h1 class="page-title">Writing</h1>
    <p class="page-subtitle">Case studies on my research projects and technical deep-dives on ML systems, atmospheric modeling, HPC workflows, and research engineering.</p>
  </div>

  {% if site.posts.size > 0 %}
  <div class="post-list">
    {% for post in site.posts %}
    <article class="post-list-item">
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
  </div>

  {% else %}
  <div class="blog-empty">
    <p>Case studies and articles coming soon — check back shortly.</p>
  </div>
  {% endif %}

</section>