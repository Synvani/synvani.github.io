---
published: true
title: Kickstarting Chirpy with a Fork
author: synvan
date: 2024-11-15 14:10:00 +0200
categories: [Blogging, Jekyll]
tags: [Chirpy, Jekyll, GitHub Pages, Blogging]
render_with_liquid: false
description: A hands-on guide to forking the Chirpy Jekyll theme and setting up your own fully customizable blog from scratch
media_subpath: '/assets/img/posts/01_kickstarting-chirpy'
image:
  path: /00_kickstarting-chirpy.jpg
  alt: 
  in_post: false
---

This post is about kickstarting a new blogging website using [Chirpy][chirpy-repo] (an awesome [Jekyll][jekyll] theme), but doing it the complicated way – forking the repository for the enhanced ability to modify features or UI. It's not recommended unless you're familiar with Jekyll (it presents plenty of challenges during upgrades), but if you're a fan of customizations and keen to learn new things – I believe it is definitely worth the journey.

If you prefer to take it easy albeit with much less customizability, you can always opt for the Starter theme – skip this post completely and read more on [Chirpy's documentation][chirpy-docs-starter]. If you wish to have full access to all the bits and bytes, continue reading here! The steps below may seem a little different compared to the official documentation, but here's how it worked for me.

## Forking the Theme

As already described in [Chirpy's documentation][chirpy-docs-fork]:

1. Sign in to GitHub.
2. [Fork the theme repository][chirpy-repo-fork].
3. Name the new repository `<username>.github.io`, replacing `username` with your lowercase GitHub username.

## GitHub Actions

1. Open your newly forked repository, which should be in: `https://github.com/<username>/<username>.github.io`
2. Navigate to the <kbd>Settings</kbd> tab.
3. Click on <kbd>Pages</kbd> in the left-hand menu.
4. Under `Build and Deployment` section,  change the `Source` to **GitHub Actions**.
   ![Build source](10_gh-actions_source.png){: .normal w='320' }
5. Navigate to the <kbd>Actions</kbd> tab and click to enable workflows.
   <!-- markdownlint-capture -->
   <!-- markdownlint-disable -->
   > Since the repository already contains workflows (written by someone else), GitHub automatically disables them for security measures. It is good practice to review workflows before you run them.
   {: .prompt-info }
   <!-- markdownlint-restore -->
   ![Workflows activation](20_gh-actions_enable-workflows.png){: .normal }

## Modifications to **config.yml**

1. Within GitHub files browser, open the `_config.yml`{: .filepath} file, it should be located in the root directory of your repository.
2. Click to edit the file and change the `url` section – it should be `https://<username>.github.io`, note that it does not end with a front-slash `/`.
    ```yaml
    # Fill in the protocol & hostname for your site.
    # E.g. 'https://username.github.io', note that it does not end with a '/'.
    url: "https://synvani.github.io"
    ```
    {: file='_config.yml'}
3. Change the `title` to reflect the name of your new website/blog, the rest can be left as is for now, as we just want to get the website up and running to make sure everything works smoothly.
    ```yaml
    title: Synvan # the main title
    ```
    {: file='_config.yml'}
4. Commit the changes made to `_config.yml`{: .filepath}, make sure to follow the [commitlint rules][commitlint-rules] otherwise your commit would fail – write something along the lines of `style: update to _config.yml`.
5. You'll notice that once the commit is accepted and applied, nothing really happens and your website is not up and online just yet (reminder, your `url` should be `https://<username>.github.io`). That's because the workflow to **Deploy and Build** the website isn't part of our available workflows!
   ![Workflows missing](30_gh-actions_build-n-deploy-missing.png){: .normal w='300'}

## Adding Deployment Workflow

1. Within GitHub files browser, navigate to `.github` directory, then `workflows`, then `starter`.
2. Inside `starter` directory you'll find `pages-deploy.yml`{: .filepath} – the missing workflow!
3. Move or copy `pages-deploy.yml`{: .filepath} file one directory up, so it is within the `workflows` directory.
   - Move file – refer to [GitHub Docs][github-docs-move-file] to learn how to move a file via GitHub's web interface.
   - Copy file – can simply download `pages-deploy.yml`{: .filepath} to your local machine and then upload it to `starter` directory.
4. Don't forget to write a valid commit according to [commitlint rules][commitlint-rules], for example: _fix: move pages-deploy.yml to workflows directory_
5. You'll notice that our commit failed – that's because our website is missing critical files that need to be compiled by us. This is mentioned in [Chirpy's FAQ][chirpy-faq-upgrade], although it may not be super clear to non-Jekyll experts where exactly this part should be added or performed...
   ![Build failed](40_build-n-deploy-failed.png){: .normal }

## Adding **npm** Install & Build Stuff

1. Navigate to `.github/workflows` directory and open `pages-deploy.yml`{: .filepath}.
2. Edit the file – the following lines should be added:

```yaml
  - name: npm build
    run: npm install && npm run build
```
{: file='pages-deploy.yml' .nolineno }

The new lines should be placed between the `name: Setup Ruby` and `name: Build site` blocks, as below:

```yaml
  - name: Setup Ruby
    uses: ruby/setup-ruby@v1
    with:
      ruby-version: 3.3
      bundler-cache: true

  - name: npm build
    run: npm install && npm run build

  - name: Build site
    run: bundle exec jekyll b -d "_site${{ steps.pages.outputs.base_path }}"
    env:
      JEKYLL_ENV: "production"
```
{: file='pages-deploy.yml' .nolineno }

Finally, commit the changes and don't forget to write a valid commit message, for example: _fix: adding npm compilation to pages-deploy.yml_`

## Checking Out Our Website

A few minutes after our last commit you should be able to see your new website proudly deployed to `https://<username>.github.io`! 

The next step is to create a local development environment so you could start working on your website conveniently and efficiently. Chirpy's documentation has a great section on that which you can follow-up on [here][chirpy-docs-dev-env]. Good luck!



[chirpy-repo]: https://github.com/cotes2020/jekyll-theme-chirpy
[jekyll]: https://jekyllrb.com/
[chirpy-docs-starter]: https://chirpy.cotes.page/posts/getting-started/#option-1-using-the-starter-recommended
[chirpy-docs-fork]: https://chirpy.cotes.page/posts/getting-started/#option-2-forking-the-theme
[chirpy-repo-fork]: https://github.com/cotes2020/jekyll-theme-chirpy/fork
[commitlint-rules]: https://commitlint.js.org/reference/rules.html
[github-docs-move-file]: https://docs.github.com/en/repositories/working-with-files/managing-files/moving-a-file-to-a-new-location
[chirpy-faq-upgrade]: https://github.com/cotes2020/jekyll-theme-chirpy/wiki/Upgrade-Guide#upgrade-the-fork
[chirpy-docs-dev-env]: https://chirpy.cotes.page/posts/getting-started/#setting-up-the-environment



