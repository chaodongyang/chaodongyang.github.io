---
layout: page
title: Wiki
description: ��ŮͼƬ
keywords: ��Ů, ��ŮͼƬ
comments: false
menu: ��Ů
permalink: /celebrity/
---

> �ô�����ô�������۰�

<ul class="listing">
{% for wiki in site.wiki %}
{% if wiki.title != "Wiki Template" %}
<li class="listing-item"><a href="{{ wiki.url }}">{{ wiki.title }}</a></li>
{% endif %}
{% endfor %}
</ul>