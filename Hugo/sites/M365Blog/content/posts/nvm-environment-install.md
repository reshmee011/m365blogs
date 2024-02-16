---
title: "NVM SPFx environment setup issues"
date: 2024-02-08T16:46:04Z
tags: ["SharePoint","SPFx","NVM","Node","Development SetUp"]
featured_image: '/posts/images/nvm-environment-spfx-setup/npm_deprecatedFeatures.png'
draft: true
---

# SPFx set using NVM to the latest node

I have used my dev machine for SPFx development for over a year and before starting I went through the normal step to install the latest node version supported by the latest SPFX version 1.18.2 using nvm 

```dotnetcli
nvm install 18.19.0
```

I went ahead to install the SPFX toolchain 

```dotnetcli
npm install gulp-cli yo @microsoft/generator-sharepoint --global
```

However the install failed with warnings

![Warnings and errors during set up](../images/nvm-environment-spfx-setup/npm_deprecatedFeatures.png)

thumbnail_logo.png


https://stackoverflow.com/questions/68260784/npm-warn-old-lockfile-the-package-lock-json-file-was-created-with-an-old-version
npm WARN old lockfile The package-lock.json file was created with an old version of npm

npm list -g  --depth=0

How to address


![folder where NVM is installed](../images/nvm-environment-spfx-setup/NVM_EnvironmentVar.png)

- Uninstall NVM and any node versions

- install nvm 
You can download the NVM package from [NVM releases](https://github.com/coreybutler/nvm-windows/releases) or use a chocolatery 



## References

[NVM releases](https://github.com/coreybutler/nvm-windows/releases)
[Set up your SharePoint Framework development environment](https://learn.microsoft.com/en-us/sharepoint/dev/spfx/set-up-your-development-environment)