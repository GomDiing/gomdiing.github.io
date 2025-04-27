---
title: "미니 프로젝트"
permalink: categories/miniProject
layout: category
author_profile: true
sidebar_main: true
---

프로젝트 카테고리 모음집입니다

{% assign posts = site.categories.miniProject %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}