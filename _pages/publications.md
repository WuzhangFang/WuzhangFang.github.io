---
layout: page
permalink: /publications/
title: publications
description: Most recent first.
nav: true
nav_order: 1
---

<!-- _pages/publications.md -->

See also my [Google Scholar profile](https://scholar.google.com/citations?user=CHP3KbgAAAAJ).

{% include bib_search.liquid %}

<div class="publications">

<h2 class="bibliography-section">journal articles &amp; preprints</h2>
{% bibliography --query @article %}

<h2 class="bibliography-section">thesis</h2>
{% bibliography --query @phdthesis %}

<h2 class="bibliography-section">conference talks &amp; proceedings</h2>
{% bibliography --query @inproceedings %}

</div>
