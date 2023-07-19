---
title: "Make Netlify site crawable by Search Engines"
date: 2023-07-18T08:29:27+01:00
draft: true
---

# Make Netlify site crawable by Search Engines

There are numerous posts to describe how to convert Wordpress to Hugo. I followed the instructions from blog post [How to move your blog from Wordpress to Hugo](https://www.m365princess.com/blogs/move-blog-wordpress-hugo/) to do mine. I had few issues with imported content from WordPress like special characters in the tags, e.g. C# which was preventing my netlify site from building , so worth reviewing the generated posts.

Fast forward few months after setting my Hugo site, my husband was searching for one of the article I told him I wrote from Google and mentioned he can't find my post and realised my site is not picked up by search engines. Paul Bullock guided me towards  custom domain. I lacked creativity and ended getting a domain based on my name. I waited for a week and checked for any updates. Alas my site was not being crawled still. Again Paul Bullock kindly pointed to the SEO/Search tools e.g. https://developers.google.com/search and https://bing.com/webmasters/about to ensure my site was registered with search engines and more troubleshooting learning more about the new set up I had with Hugo.
# Issue with Ananke theme

My site was not being crawled due to the theme Ananke I was using. Please find more info from [How to configure robots(search engines) to follow and to index?](https://discourse.gohugo.io/t/how-to-configure-robots-search-engines-to-follow-and-to-index/28449/5) The theme uses input from robots.txt is in /themes/ananke/layouts/robots.txt which has the following code:

```tex
User-agent: *
{{/* robotstxt.org - if ENV production variable is false robots will be disallowed. */ -}}
{{ if eq (getenv "HUGO_ENV") "production" | or (eq .Site.Params.env "production")  -}}
Allow: /
Sitemap: {{ "/sitemap.xml" | absURL }}
{{ else -}}
Disallow: /
{{ end -}}
```

I needed to figure how to set the environment to production to allow the robotstxt
I found the solution from blog post (Getting started with Hugo)[https://davidrothera.me/posts/getting-started-with-hugo/] covers how to resolve it by adding a custom netlify.toml config to avoid going to the settings on netlify 

```tex
# netlify.toml

[build]
command = "hugo --gc --minify -b $URL"
publish = "public"

[build.environment]
NODE_ENV = "production"
GO_VERSION = "1.19.4"
TZ = "Europe/Dublin"    # Set to preferred timezone

[context.production.environment]
HUGO_VERSION = "0.109.0"
HUGO_ENV = "production"

[context.deploy-preview.environment]
HUGO_VERSION = "0.109.0"
```
# Add site map to Google

To help my site and page to be picked up I decided to do the following additional steps.

- Navigate to https://developers.google.com/search

- Enter my domain to check
![Add Domain](../images/netlifySiteCrawable/AddDomain.png)

- Add CNAME record to domain settings within Netlify 
![Add Domain CNAME](../images/netlifySiteCrawable/DomainCName.png)

- Add sitemap.xml from site,  it can be found at /sitemap.xml
![Add SiteMap](../images/netlifySiteCrawable/AddSiteMap.png)

- Indexing Page
![Indexing Page](../images/netlifySiteCrawable/IndexingPage.png)

Waiting for at least a day to see the results
https://app.netlify.com/teams/reshmee011/dns/reshmeeauckloo.com

However https://www.google.com/webmasters/tools/robots-testing-tool
