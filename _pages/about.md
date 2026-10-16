---
layout: about
title: about
permalink: /
subtitle: Postdoctoral Researcher · Ping Group · University of Wisconsin–Madison

profile:
  align: right
  image: prof_pic.jpg
  image_circular: true # crops the image to make it circular

selected_papers: true # includes a list of papers marked as "selected={true}"
social: true # includes social icons at the bottom of the page

announcements:
  enabled: false # includes a list of news items (also uncomment the news collection in _config.yml)
  scrollable: true # adds a vertical scroll bar if there are more than 3 news items
  limit: 5 # leave blank to include all the news in the `_news` folder

latest_posts:
  enabled: false
---

I am a computational condensed-matter physicist working in Prof. Yuan Ping's group at the University of Wisconsin–Madison. I develop and apply **first-principles methods** to understand how spin, charge, and lattice degrees of freedom interact in magnetic materials for spintronic devices.

My current research focuses on:

- **Magnon–phonon coupling** — efficient first-principles methods for computing how spin waves and lattice vibrations interact, especially in two-dimensional van der Waals magnets.
- **Spin–orbit torque and spin transport** — how current-induced torques arise in magnetic bilayers and low-symmetry materials, and what they mean for magnetization switching.
- **Magnetism in low-dimensional materials** — non-collinear order, spin spirals and skyrmions, and magnetic impurities in Ising superconductors.
- **Ultrafast spin dynamics** — spin non-collinear real-time time-dependent density-functional theory (RT-TDDFT), implemented in the GPU-accelerated [INQ](https://gitlab.com/npneq/inq) code.

I received my Ph.D. in physics from the University of Nebraska–Lincoln in 2023, where I worked with Prof. Kirill Belashchenko on first-principles calculations of voltage-controlled magnetic anisotropy and spin–orbit torque. Before that, I studied at the University of Chinese Academy of Sciences.

Besides research papers, I write [**tutorials**]({{ '/tutorials/' | relative_url }}) on the computational tools and methods.

{% assign recent_tutorials = site.tutorials | sort: "date" | reverse %}
{% if recent_tutorials.size > 0 %}

<h2><a href="{{ '/tutorials/' | relative_url }}" style="color: inherit">latest tutorials</a></h2>

<ul class="recent-tutorials">
{% for t in recent_tutorials limit: 3 %}
  <li><a href="{{ t.url | relative_url }}">{{ t.title }}</a> <span class="text-muted">— {{ t.description }}</span></li>
{% endfor %}
</ul>
{% endif %}
