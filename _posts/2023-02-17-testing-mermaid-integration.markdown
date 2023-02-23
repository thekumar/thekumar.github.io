---
layout: post
title:  "Blogging upgrades"
date:   2023-02-17 12:08:00 -0800
categories: jekyll
mermaid: true
---

One of the important drivers for migrating my blog over to Github Pages / Jekyll has been the need to create more technical articles
with ease. This meant, I wanted to be able to integrate with nifty content creation JS libraries such as ASCIInema, Mermaid and JsFiddle.

To start things off, here's the first integration: Mermaid.

<div class="mermaid">
flowchart TD;
    A[Step 1: Collect Users] -->|lots of users| B(Step 2???);
    B --> C{???};
    C -->|One| D[Profit!];
    C -->|Two| E[Profit!];
    C -->|Three| F[fa:fa-car Fast car!];
</div>

I have also updated the blog with pagination and added jsonresume integration. Check out the README.md file at [the repo](https://github.com/thekumar/thekumar.github.io)
