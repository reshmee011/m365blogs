---
title: "Creating New Posts with Hugo"
date: 2023-12-18T04:10:42Z
tags: ["Hugo", "Blogging", "Tutorial"]
featured_image: '/posts/images/Hugo_NewPosts/Post.png'
draft: false
---

# Creating New Posts with Hugo

In this post, we will walk through the process of creating a new post in Hugo, a popular static site generator.

## Step 1: Navigate to Your Hugo Site

First, navigate to the directory where your Hugo site is configured. You can do this using the `cd` command in terminal:

``` PowerShell
cd .\Hugo\sites\M365Blog\
```

## Step 2: Create a New Post

Next, use the hugo new command to create a new post:

```powershell
hugo new posts/Hugo_create-new-post.md
```

This command will create an empty Markdown file with some default Hugo metadata, such as the title, date, and draft status:

```HTML
---
title: "Hugo_create-new-post"
date: 2023-12-18T04:10:42Z
draft: true
---
```

## Step 3: Add Metadata and Content
Once you've written your post, you can add additional metadata like tags and a featured image. You can also set the draft status to false to indicate that the post is ready to be published. The available metadata fields may vary depending on the Hugo theme you're using. In this example, we're using the Ananke theme:

```PowerShell
---
title: "Creating Hugo Posts"
date: 2023-12-18T04:10:42Z
tags: ["Hugo","Posts","New"]
featured_image: '/posts/images/Hugo_NewPosts/Update.png'
draft: false
---
```


In this example:

* title: The title of the blog post.
* date: The date the blog post was created.
* tags: A list of tags associated with the blog post.
* featured_image: The path to an image that will be displayed as the featured image for the blog post.
* draft: A boolean value that indicates whether the blog post is a draft. If draft is true, the post will not be included in the build output unless the --buildDrafts flag is used when running Hugo.

### Step 4: Commit Your Changes

Finally, commit the changes to your repository to publish the post.

Happy blogging with Hugo!