---
layout: page
title: Tags
subtitle: Browse posts by tag
---

<div class="tag-cloud" style="margin-bottom: 2.5rem; line-height: 2.2;">
{% assign tag_names = site.tags | sort %}
{% for tag in tag_names %}
{% assign count = tag[1].size %}
{% assign font_size = count | times: 0.2 | plus: 0.9 %}
<a href="#tag-{{ forloop.index }}" style="font-size: {{ font_size }}em; margin-right: 0.8em; white-space: nowrap;">{{ tag[0] }}<sup>{{ count }}</sup></a>
{% endfor %}
</div>

{% for tag in tag_names %}
## {{ tag[0] }} <a id="tag-{{ forloop.index }}"></a>

<ul>
{% for post in tag[1] %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a> &mdash; {{ post.date | date: "%B %-d, %Y" }}</li>
{% endfor %}
</ul>
{% endfor %}
