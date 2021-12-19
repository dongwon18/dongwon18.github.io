---
title: "SW Engineering"
layout: archive
permalink: categories/#swengineering
author_profile: true
sidebar_main: true
---

{% include https://dongwon18.github.io/swengineering/SWEngineering_start/archive-single.html %}

{% assign posts = site.categories.#swengineering %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
