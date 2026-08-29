---
layout: page
title: Series
permalink: /series/
---

Some of what we write runs as a series — one problem per post, in the order we hit them,
with an argument that only finishes at the end. Posts from the two of us are interleaved on
the home page, so this is where a series can be read in order.

{% for entry in site.data.series %}
{% assign key = entry[0] %}{% assign meta = entry[1] %}
{% assign items = site.posts | where: "series", key | sort: "date" %}

<h2 id="{{ key }}">{{ meta.title }}</h2>

<p class="series-tagline">{{ meta.tagline }}</p>

{{ meta.description }}

{% assign who = site.data.authors[meta.author] %}{% if who %}By {{ who.name }}. {% endif %}{{ items.size }} part{% if items.size != 1 %}s{% endif %} so far.

<ol class="series-list">
{% for p in items %}
  <li>
    <a href="{{ p.url | relative_url }}">{{ p.title }}</a>
    <span class="series-list-date">{{ p.date | date: "%b %-d, %Y" }}</span>
    {% if p.subtitle %}<span class="series-list-sub">{{ p.subtitle }}</span>{% endif %}
  </li>
{% endfor %}
</ol>
{% endfor %}
