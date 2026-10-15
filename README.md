# wuzhangfang.github.io

Personal research website of Wuzhang Fang, built with [al-folio](https://github.com/alshedivat/al-folio) (Jekyll) and hosted on GitHub Pages.

Every push to `main` triggers the **Deploy site** GitHub Action, which builds the site and publishes it to the `gh-pages` branch. The live site updates about 2–4 minutes later.

## Where things live

| What                       | File(s)                                       |
| -------------------------- | --------------------------------------------- |
| Name, site URL, settings   | `_config.yml`                                 |
| Home page bio              | `_pages/about.md`                             |
| Profile photo              | `assets/img/prof_pic.jpg` (replace this file) |
| Social / contact links     | `_data/socials.yml`                           |
| Publications               | `_bibliography/papers.bib`                    |
| Tutorials                  | `_tutorials/*.md`                             |
| Tutorial notebooks         | `assets/jupyter/*.ipynb`                      |
| CV PDF (optional)          | `assets/pdf/cv.pdf` + uncomment in `socials.yml` |

## Add a publication

Paste the BibTeX entry into `_bibliography/papers.bib`. The list is grouped by year, newest first, and split into journal articles, thesis, and conference talks by entry type (`@article`, `@phdthesis`, `@inproceedings`).

Optional fields that add buttons and badges:

```bibtex
  abbr     = {PRB},                          % venue badge
  selected = {true},                         % also show on the home page
  html     = {https://journals.aps.org/...}, % "HTML" button
  pdf      = {fang2025efficient.pdf},        % "PDF" button; file goes in assets/pdf/
  arxiv    = {2501.01234},                   % "arXiv" button
  doi      = {10.1103/...},
  code     = {https://github.com/...},
  bibtex_show = {true},                      % "Bib" button
```

## Add a Markdown tutorial

Create `_tutorials/my-tutorial.md`:

```markdown
---
title: Running your first DFT calculation
description: One-sentence summary shown on the tutorials page.
date: 2026-11-01
topic: DFT basics          # tutorials are grouped by topic
format: markdown
level: Beginner            # optional
tags: [dft, quantum-espresso]
---

Your content. Code blocks, $\LaTeX$ math, tables and images all work.
```

It appears at `/tutorials/my-tutorial/`. A table of contents sidebar is added automatically. Set `toc: false` to hide it, or `draft: true` to keep the tutorial off the listing page.

## Add a notebook tutorial

1. Run the notebook so its outputs (plots, printed results) are saved, then put it in `assets/jupyter/`.
2. Copy `_tutorials/magnon-dispersion-notebook.md`, then change the title, description, `notebook:` path, and the path inside `file_exists`.

The page embeds the rendered notebook and adds **Open in Colab** and **View on GitHub** buttons.

## Preview locally (optional)

```bash
bundle install
pip install nbconvert
bundle exec jekyll serve      # http://localhost:4000
```

Or with Docker: `docker compose up`.

Template documentation for everything else is in `docs/`.
