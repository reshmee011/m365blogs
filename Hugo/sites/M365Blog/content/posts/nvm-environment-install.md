---
title: "Troubleshooting NVM Setup Issues for SPFx Development"
date: 2024-04-03T01:00:00Z
tags: ["SharePoint","SPFx","nvm","Node","Development SetUp"]
featured_image: '/posts/images/nvm-environment-spfx-setup/npm_deprecatedFeatures.png'
omit_header_text: true
draft: false
---

# Troubleshooting NVM Setup Issues for SPFx Development

I have not used my dev machine for SPFx development for over a year. Before diving into any new SPFx development, I followed the typical procedure of installing the latest Node.js version 18.19.0 supported by the current SPFx version (1.18.2) using nvm (node version management).nvm allows to maintain different development environment.

```dotnetcli
nvm install 18.19.0
```
Following this, I proceeded to install the SPFx toolchain:

```dotnetcli
npm install gulp-cli yo @microsoft/generator-sharepoint --global
```

However, this seemingly straightforward process was plagued with warnings and errors:

```dotnetcli
node. exe : npm WARN deprecated readdir-scoped-modules@1.1.0: This functionality has been moved to @npmcli/fs
At C:\Program Files\nodejs\npm. ps1:16 char :5
"$basedir/node_modules/npm/bin/npm-cli.js"
NotSpecified: (npm WARN deprec ... d to @npmcli/fs:String) [], RemoteException
+ FullyQualifiedErrorId : NativeCommandError

npm WARN deprecated debuglog@1.0.1: Package no longer supported. Contact Support at https://www.npmjs.com/support for more info.
...
```

![Warnings and errors during set up](../images/nvm-environment-spfx-setup/npm_deprecatedFeatures.png)

Subsequently, attempts to use the yo commandlet failed:

![node yo SharePoint errors](../images/nvm-environment-spfx-setup/node-yo-sharepoint-error.png)

```dotnetcli
node. exe :
At C: \Program Files\nodejs\yo.ps1:16 char :5
"$basedir/node_modules/yo/lib/cli.js" $arg ...

: NotSpecified: (:String) [], RemoteException
+ FullyQualifiedErrorId : NativeCommandError
```

Additionally, installing gulp-cli produced errors:

```dotnetcli
PS C: \windows\system32> node -- version
v18.19.0

PS C: \windows\system32> npm install gulp-cli
node. exe : npm WARN old lockfile
At C: \Program Files\nodejs\npm.ps1:16 char :5
"$basedir/node_modules/npm/bin/npm-cli.js"
& "Sbasedir/nodeSexe"
```
 
![npm lock file](../images/nvm-environment-spfx-setup/npmwarnoldlockfile.png). Refer to [npm WARN old lockfile The package-lock.json file was created with an old version of npm](https://stackoverflow.com/questions/68260784/npm-warn-old-lockfile-the-package-lock-json-file-was-created-with-an-old-version) for more info.


## npm list

I used the command ```npm list -g --depth=0``` for troubleshooting. It is used to list globally installed npm packages without displaying their dependencies. It's a useful way to quickly view what npm packages you have installed globally on your system.

![npm list g](../images/nvm-environment-spfx-setup/npm-list-g.png)

## nvm locations

By default nvm is installed at users **AppData\Roaming\nvm** for users
![npm default install](../images/nvm-environment-spfx-setup/nvm_default_install.png)

However, depending on machine setup, it's essential to confirm the location by checking the environment variables for NVM_HOME
![folder where NVM is installed](../images/nvm-environment-spfx-setup/NVM_EnvironmentVar.png)

## Solution

To address the myriad of issues encountered during the SPFx upgrade, the ultimate solution was to uninstall NVM and all associated node versions. 

- Uninstall NVM and any node versions
![uninstall](../images/nvm-environment-spfx-setup/nvm_uninstall.png)

- install nvm 
You can download the NVM package from [NVM releases](https://github.com/coreybutler/nvm-windows/releases) to install it, configure node and install the SPFx as per [Set up your SharePoint Framework development environment](https://learn.microsoft.com/en-us/sharepoint/dev/spfx/set-up-your-development-environment?wt.mc_id=MVP_308367)

Successful installation of yo
![npm version](../images/nvm-environment-spfx-setup/npminstallgulpcliyo.png)

## Conclusion

To ensure a smooth development experience, it is recommended to keep your environment up to date. Incompatibility issues can often arise from using outdated versions of components. Therefore, it is advisable to regularly update and reconfigure your environment from scratch to mitigate any potential issues.

## References

[NVM releases](https://github.com/coreybutler/nvm-windows/releases)

[Set up your SharePoint Framework development environment](https://learn.microsoft.com/en-us/sharepoint/dev/spfx/set-up-your-development-environment?wt.mc_id=MVP_308367)
