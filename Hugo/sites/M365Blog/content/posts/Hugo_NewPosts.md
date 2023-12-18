---
title: "Hugo_NewPosts"
date: 2023-12-18T04:10:42Z
tags: ["Hugo","Posts","New"]
featured_image: '/posts/images/Hugo_NewPosts/Update.png'
draft: true
---

Navigate to the correct path where Hugo has been configured

```powershell
cd .\Hugo\sites\M365Blog\
```

Run hugo new to create a new post

```powershell
hugo new posts/DocumentIntelligenceStudio_PreviousModelsAPIVersions.md
```

By default it creates an empty file with some Hugo Metadata like title, date and draft

```HTML
---
title: "Hugo_NewPosts"
date: 2023-12-18T04:10:42Z
draft: true
---
```

Once the post is ready to be published add additional metadata like tags, featured_image and set draft to false. The additional metedata are driven by the Hugo theme. In my instance I am using the ananke theme.


```dotnetcli
---
title: "Hugo_NewPosts"
date: 2023-12-18T04:10:42Z
tags: ["Hugo","Posts","New"]
featured_image: '/posts/images/Hugo_NewPosts/Update.png'
draft: false
---
```

Check in the changes to your repository.