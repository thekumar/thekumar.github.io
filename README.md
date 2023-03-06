# thekumar.github.io

This repo contains the customizations I have made to a standard Jekyll blog to get it to work to a satisfactory level with Github Pages.

Github Pages prefers to use the `minima` theme by default. So, my blog uses that as well as I prefer a minimalistic theme as well.
But, minima comes with its own little list of quirks and lacks some basic functionality we've all come to expect from a minimal blog.

For instance, the default layout is found under home.html. And it doesn't paginate your blog posts by default.
Github Pages has limited support for Jekyll plugins, and sadly only supports v1.1 of `jekyll-paginate`, while all the useful features are
only found in `jekyll-paginate-v2`. So, suffice it to say, I have made plenty of customizations in this repo to support the following features:

1. Paginate posts with a pagination nav bar. (See `_layouts/home.html` and `_includes/blog_nav.html`)
2. Speaking of Categories and Tags, I have a commented out tags page `_layouts/tags.html` and `_includes/tags_list.html`.
3. Integrate with mermaidjs `_includes/mermaid.html`, asciinema `_includes/asciinema.html`, and highlightjs `_includes/highlight.html`. (Also see `_includes/head.html`)
4. I've also added a Github workflow (`.github/workflows/export-resume.yml`) to auto generate my Resume in HTML and PDF formats whenever my `resume.json` file changes.
  - [![Publish Resume](https://github.com/thekumar/thekumar.github.io/actions/workflows/export-resume.yml/badge.svg)](https://github.com/thekumar/thekumar.github.io/actions/workflows/export-resume.yml)
