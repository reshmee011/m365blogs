---
title: "How to Ensure Your Netlify Site is Search Engine Crawlable"
date: 2023-07-18T08:29:27+01:00
draft: false
tags: ["Hugo", "Netlify","Robot.txt","Search Engine"]
---

# How to Ensure Your Netlify Site is Search Engine Crawlable

There are numerous posts to describe how to convert Wordpress to Hugo. I followed the instructions from blog post [How to move your blog from Wordpress to Hugo](https://www.m365princess.com/blogs/move-blog-wordpress-hugo/) to start with and hosted it on Netlify, it's essential to ensure that search engines can crawl and index your site properly. Follow these steps to make sure your site is discoverable:

## Review Content Issues

After migrating your content from WordPress to Hugo, check for any potential issues with imported content. Special characters in tags, like "C#," might cause problems during the site build process. Review the generated posts and fix any such issues.

## Custom Domain 

[Paul Bullock](https://twitter.com/pkbullock) guided me towards custom domain. I lacked creativity and ended getting a domain based on my name. Alas my site was not being crawled after one week. 

## Register with Search Engines

 [Paul Bullock](https://twitter.com/pkbullock) pointed to the SEO/Search tools e.g. https://developers.google.com/search and https://bing.com/webmasters/about to ensure my site was registered with search engines.

## Submit sitemap to Google

A sitemap provides a list of all the pages on your site, helping search engines understand its structure and content better. With Hugo, a sitemap.xml file is generated and follow the steps below to submit to Google

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

## Identify any configuration which is preventing your site from being crawled 

My site was not being crawled due to the theme Ananke I was using. The theme uses input from robots.txt  in /themes/ananke/layouts/robots.txt which has the following code:

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

The environment needs to be set to production during build to allow the robots.  I found the solution from blog post (Getting started with Hugo)[https://davidrothera.me/posts/getting-started-with-hugo/] and I added a custom netlify.toml config to avoid going to the settings on netlify to set the build environment to Production.

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

## Monitor to ensure there are no more issues
 
While the process may take some time to show results, regularly monitoring your site's performance using tools like Google Search Console will help you identify and resolve any remaining issues.  I had the warning ![Indexed, Though Blocked by robots.txt](../images/netlifySiteCrawable/IndexedThoughBlockedByRobotstxt.png) from [Google Search console](https://search.google.com/search-console) the next day and did more troubleshooting with blog post [Finding the Source of the 'Indexed, Though Blocked by robots.txt' Error](https://kinsta.com/knowledgebase/indexed-though-blocked-by-robots-txt/). I went to [Google’s robots.txt tester](https://www.google.com/webmasters/tools/robots-testing-tool) and tested one URL and seems there were no issues.  

![URL Allowed](../images/netlifySiteCrawable/URLAllowed.png)

After few days of issue 'Indexed, Though Blocked by robots.txt' fixed, the pages were not indexed with "Discovered – currently not indexed" and "Page with redirect" as reasons not being indexed from https://search.google.com/search-console. 

![Pages Not Indexed](../images/netlifySiteCrawable/PageRedirectNoIndexedIssues.png)

I could not find any solutions to the two messages above and hoping by posting my articles on social media might force Google to index my site.

Patience is key, as it may take a few days for search engines to fully crawl and index your site.
## Conclusion

Ensuring that your Netlify site is crawlable by search engines is vital for its visibility and discoverability on the internet. By following the steps outlined in this guide, you can overcome potential obstacles and improve your site's indexing and ranking. 