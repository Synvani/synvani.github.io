---
published: false
title: Custom Domain for Jekyll
author: synvan
date: 2024-07-20 14:10:00 +0200
categories: [Blogging, Jekyll]
tags: [Chirpy, Jekyll, GitHub Pages]
render_with_liquid: false
description: 
media_subpath: '/assets/img/posts/02_custom-domain'
---

Setting up a custom domain for your [Jekyll][jekyll] website is a simple way to give a more professional and polished look. Instead of a generic address like `synvan.github.io`, you can switch to something cleaner like `synvan.com` or `synvan.io`. In this post, we'll go through how to set that up step by step.

## Domain Purchase

First action would be to buy and own a domain. I recommend [Cloudflare][cloudflare-home] since it is a reputable internet service provider which basically offers registering domains through them at cost value. Also, it has dark mode!

1. Go to [Cloudflare][cloudflare-home] and create an account – it's extremely convenient to simply login with your Google/Apple account (I assume you have at least one of those – one less password to remember).
2. Once logged in, navigate to <kbd>Domain Registration</kbd> in the left-hand menu, and then slide into the <kbd>Register Domains</kbd> sub-menu.
3. From here on it's pretty straightforward – flow with your imagination and check availability and prices for your favorite domain names.
4. Purchase the domain name!

## Verify Your Domain

Verifying a custom domain with GitHub ensures that only repositories owned by you can publish to that domain (and its subdomains). This protects against domain hijacking, which can occur if a repository is deleted or GitHub Pages is disabled while the domain remains active but unverified. Once verified, the domain and its subdomains are safe from shady actions by other GitHub users.

GitHub Docs already have a super clear and detailed [guide][github-docs-verify-domain] on how to verify your domain via your account settings on GitHub.

Here's how to find the DNS management page on Cloudflare's dashboard:
1. Open your [dashboard][cloudflare-dash].
2. Navigate to <kbd>Domain Registration</kbd> in the left-hand menu, and then slide into <kbd>Manage Domains</kbd>.
3. Click on `Manage` in the row of the domain name you just purchased.
4. In the right-hand `Quick Actions` menu, click on `Update DNS configuration`.
   1. As instructed on [GitHub Docs guide][github-docs-verify-domain], in Cloudflare's DNS Records dashboard click on <kbd>Add record</kbd> (located to the right of the DNS records search bar).
   2. Make sure the DNS Record type is set to `TXT`.
   3. Record's `Name` should be copied from GitHub's instructions, don't forget to prefix with your domain name (e.g., `synvan.com`).
   4. Record's `Content` should be the code value given by GitHub.

Once domain verification is complete it should look like this:
![Domain verified](01_verified-domain.png){: .normal }

## Configuring an Apex Domain

Apex domains and subdomains are both parts of a website's address, but they serve different purposes. An apex domain, also known as a root domain, is the main website address (e.g., `synvan.com`). A subdomain is an additional part of the address that comes before the apex domain (e.g., `blog.synvan.com`).

I assume you're keeping it simple and just want to configure an apex (root) domain, GitHub Docs has a [great guide][github-docs-apex-domain] with detailed explanations step-by-step. In a nutshell the process can be summarized into two steps:
1. **GitHub Repository Settings:** Plugging in the new custom domain name.
2. **Domain Management Dashboard**: Setting up the DNS records.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Shortly after typing in your custom domain name and hitting _Save_, a DNS check by GitHub will be triggered. It will likely fail since you probably did not manage to complete the DNS records setup yet, and even if you did (nice!) – DNS updates sometime take awhile to apply.
{: .prompt-info }
<!-- markdownlint-restore -->
![DNS check](02_dns-check.png){: .normal }

**Quick recommendation regarding DNS records:**
- If your DNS provider supports `ALIAS` or `ANAME` at the root, go with that as it’s lower maintenance (unfortunately, Cloudflare does not support `ALIAS`/`ANAME` at this point).
- Otherwise, create four `A` records pointing to GitHub’s IPv4 addresses.
- If you want IPv6 support, also add the `AAAA` records GitHub recommends, but keep the `A` records due to spotty IPv6 adoption.

**Here's a more detailed breakdown:**
- `A` Record: 
  - **Function:** Maps a domain name (like `synvan.com`) to an IPv4 address (like `185.199.108.153`).
  - **Use Case:** Directly points a domain name to the server hosting the website or service.
  - **Limitations:** Only works with IPv4 addresses. Requires an `AAAA` record for IPv6.
- `CNAME` Record:
  - **Function:** Maps a domain name to another domain name, essentially creating an alias.
  - **Use Case:** Useful for creating aliases like `www.synvan.com` pointing to `synvan.com` or for pointing to services hosted on other domains.
  - **Limitations:** Cannot be used at the root of a domain (apex) like `synvan.com`. It also cannot coexist with other records on the same domain name.
- `ALIAS`/`ANAME` Record: 
  - **Function:** Provides a way to point a domain name (including the apex) to another domain name, but internally resolves it to an `A` or `AAAA` record. 
  - **Use Case:** Useful for pointing the root domain (e.g., `synvan.com`) to another domain (e.g., a cloud service) while still allowing other records on the same domain. 
  - **How it works (simplified):** When a DNS server receives a query for an `ALIAS` record, it looks up the `A` or `AAAA` records of the target domain and returns those as the answer, effectively hiding the underlying IP address change from the user. 
  - **Availability:** Not a standard DNS record type, but offered by some DNS providers (not by Cloudflare).






[jekyll]: https://jekyllrb.com/
[cloudflare-home]: https://www.cloudflare.com/
[github-docs-verify-domain]: https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages
[cloudflare-dash]: dash.cloudflare.com
[github-docs-apex-domain]: https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain

