---
published: true
title: Starting with Chirpy the Hard Way
author:
date: 2024-07-20 14:10:00 +0200
categories: [Blogging, Tutorial]
tags: [Chirpy, Jekyll, GitHub Pages]
render_with_liquid: false
description: 
---

This post is about kickstarting a new blogging website using [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy), an awesome [Jekyll](https://jekyllrb.com/) theme, but doing it the complicated way - forking the repository for the enhanced ability to modify features or UI. It's not recommended unless you're familiar with Jekyll since it presents plenty of challenges during upgrades, but if you're a fan of customizations and also keen to learn new things, I believe it definitely is worth the journey.

## Forking the Theme

As already described in [Chirpy's documentation](https://chirpy.cotes.page/posts/getting-started/#option-2-forking-the-theme):

1. Sign in to GitHub.
2. [Fork the theme repository](https://github.com/cotes2020/jekyll-theme-chirpy/fork).
3. Name the new repository `<username>.github.io`, replacing `username` with your lowercase GitHub username.

## GitHub Actions

1. Open your newly forked repository, which should be in `https://github.com/<username>/<username>.github.io`.
2. Go to _Settings_ tab -> _Pages_ -> change deployment method to _GitHub Actions_.
3. Go to _Actions_ tab and enable workflows.

## Quick Modifications to __config.yml__

1. Open the `_config.yml`{: .filepath} file, it should be located in the root directory of your repository.
2. Change the `url` - it should be like `https://username.github.io`, note that it does not end with a '/'.
3. Change the title to reflect your new website\blog, the rest can be left as is for now, as we just want to get the website up and running to make sure everything works smoothly.

## Moving __pages-deply.yml__ to _workflows_ Folder

...

## Editing __pages-deply.yml__ to Install & Build __npm__ Stuff

...




