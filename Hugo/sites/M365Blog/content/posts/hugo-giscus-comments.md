To add Giscus, a commenting system powered by GitHub Discussions, to a Hugo blog post, follow these steps:

Register Giscus on Your GitHub Repository:

Go to the Giscus App and sign in with your GitHub account.
Select the repository you want to use for storing comments.
Configure the settings as per your preference (e.g., category, reactions).
Copy the generated script from Giscus.
Integrate Giscus Script into Your Hugo Site:

Create a partial template in your Hugo site if you don't already have one for custom scripts. Typically, this would be under /layouts/partials/.
Paste the Giscus script you copied into this new partial file. You might name the file something like giscus.html.

<!-- layouts/partials/giscus.html -->
<script src="https://giscus.app/client.js"
    data-repo="username/repo"
    data-repo-id="repo-id"
    data-category="Announcements"
    data-category-id="category-id"
    data-mapping="pathname"
    data-reactions-enabled="1"
    data-emit-metadata="0"
    data-input-position="top"
    data-theme="preferred_color_scheme"
    data-lang="en"
    crossorigin="anonymous"
    async>
</script>

Include the Partial in Your Blog Post Layout:

Find the Hugo layout file that renders your blog posts. This might be something like /layouts/posts/single.html.
Include the giscus.html partial in this layout where you want the comments to appear, usually at the end of the blog post content.

{{ .Content }}


<!-- Include Giscus comments -->
{{ partial "giscus.html" . }}


Adjust Your Content Security Policy (CSP):

If your site uses a Content Security Policy (CSP), ensure it allows scripts from https://giscus.app.
Test the Integration:

Deploy your changes to your live site.
Open a blog post and verify that the Giscus commenting system loads correctly.
Remember to replace placeholders like username/repo, repo-id, and category-id with the actual values provided by Giscus when you registered your repository.