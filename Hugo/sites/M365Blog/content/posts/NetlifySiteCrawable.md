---
title: "Make Netlify site crawable by Search Engines"
date: 2023-07-18T08:29:27+01:00
draft: true
---

# Make Netlify site crawable by Search Engines

There are numerous posts to describe how to convert Wordpress to Hugo. I followed the instructions from blog post to convert my Wordpress to Hufo [How to move your blog from Wordpress to Hugo](https://www.m365princess.com/blogs/move-blog-wordpress-hugo/). I had a few issues like special characters in the tags, e.g. C# which was preventing my netlify site from building. 

After a few months of setting my Hugo site, my husband was searching for one of the article I told him I wrote from Google and mentioned he can't find my post. 

I never thought of searching for my own content from search engine and realise none of my posts are searchable.

# Identify Issue with Ananke theme
My site was not being crawled and nailed it down to the theme I was using. I found the solution from blog post (Getting started with Hugo)[https://davidrothera.me/posts/getting-started-with-hugo/] covers how to resolve it through the custom netlify.toml config file as the theme itself can't be updated. 

# Add netlify.toml 

Add netlify.toml as per blog post 

# Add site map to Google
https://search.google.com/search-console/about


https://reshmeeauckloo.com/sitemap.xml

![Add SiteMap](../images/netlifySiteCrawable/AddSiteMap.png)

# Add CNAME record to domain settings

https://app.netlify.com/teams/reshmee011/dns/reshmeeauckloo.com
