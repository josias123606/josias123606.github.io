---
layout: default
title: Entradas
---

<h2>📝 Todas las Entradas</h2>
<p style="color:#888; margin-bottom:24px;">Explora todos los artículos del blog organizados por fecha. Usa las categorías de la derecha para filtrar.</p>

{% for post in site.posts %}
<article class="blog-post" data-categories="{{ post.categories | join: ', ' }}">
    <h3><a href="{{ post.url }}" style="color:#f0f0f0; text-decoration:none;">{{ post.title }}</a></h3>
    <div class="blog-post-meta">
        <span>{{ post.date | date: "%d de %B, %Y" }}</span>
        {% for category in post.categories %}
            <span class="blog-post-category">{{ category }}</span>
        {% endfor %}
    </div>
    <p>{{ post.excerpt | strip_html | truncatewords: 35 }}</p>
    <a href="{{ post.url }}" style="color:#888; text-decoration:none; font-size:12px; font-weight:500;">Leer más →</a>
</article>
{% endfor %}

{% if site.posts.size == 0 %}
<p style="color:#888;">Aún no hay entradas publicadas.</p>
{% endif %}
