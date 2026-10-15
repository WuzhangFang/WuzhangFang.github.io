---
title: Plotting magnon dispersions with NumPy
description: A runnable notebook that plots spin-wave dispersions for square and triangular ferromagnets and checks the Dk² limit.
date: 2026-10-15
topic: Magnons & spin waves
format: notebook
level: Beginner
tags: [magnons, python, jupyter]
notebook: assets/jupyter/magnon-dispersion.ipynb
toc: false
---

{% assign nb_repo_path = site.github_repo | append: "/blob/" | append: site.github_branch | append: "/" | append: page.notebook %}

<p>
  <a class="btn btn-sm z-depth-0" role="button" href="https://colab.research.google.com/github/{{ nb_repo_path }}" target="_blank" rel="noopener">
    <i class="fa-solid fa-play"></i> Open in Colab
  </a>
  <a class="btn btn-sm z-depth-0" role="button" href="https://github.com/{{ nb_repo_path }}" target="_blank" rel="noopener">
    <i class="fa-brands fa-github"></i> View / download on GitHub
  </a>
</p>

{::nomarkdown}
{% assign jupyter_path = page.notebook | relative_url %}
{% capture notebook_exists %}{% file_exists assets/jupyter/magnon-dispersion.ipynb %}{% endcapture %}
{% if notebook_exists == "true" %}
  {% jupyter_notebook jupyter_path %}
{% else %}
  <p>Sorry, this notebook could not be found.</p>
{% endif %}
{:/nomarkdown}
