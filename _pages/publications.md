---
layout: page
permalink: /publications/
title: publications
# description: Selected publications in reversed chronological order.
years: [2024, 2023, 2021, 2018, 2013, 2010, 2009, 2008, 2007, 2005]
nav: true
nav_order: 1
---
<!-- _pages/publications.md -->
Selection of my favourite publications. A full list can be found at my [scholar profile](https://scholar.google.com/citations?user=gJlLSIEAAAAJ).

<div class="publications">

{%- for y in page.years %}
  <h2 class="year">{{y}}</h2>
  {% bibliography -f {{ site.scholar.bibliography }} -q @*[year={{y}}]* %}
{% endfor %}

</div>
