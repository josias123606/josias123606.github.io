---
layout: default
title: Blog
---

<h2>Últimas Entradas</h2>

{% for post in site.posts %}
<article class="blog-post" data-categories="{{ post.categories | join: ', ' }}">
    <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
    <div class="blog-post-meta">
        <span>{{ post.date | date: "%d de %B, %Y" }}</span>
        {% if post.categories %}
            {% for category in post.categories %}
                <span class="blog-post-category">{{ category }}</span>
            {% endfor %}
        {% endif %}
    </div>
    
    <p>{{ post.excerpt | strip_html | truncatewords: 30 }}</p>
    <a href="{{ post.url }}" style="color: #888; text-decoration: none; font-size: 12px; font-weight: 500;">Leer más →</a>
</article>
{% endfor %}