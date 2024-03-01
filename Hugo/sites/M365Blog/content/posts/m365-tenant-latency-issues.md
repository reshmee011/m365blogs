---
title: "M365 Tenant Latency Issues"
date: 2024-02-29T10:56:54Z
draft: true
---

# Latency issues accessing MS sites - SharePoint, Admin Centre 


## Delays loading CDN files

https://res-1.cdn.office.net/files/odsp-web-prod_2024-02-09.011/splistwebpack/1302.js
GET https://res-1.cdn.office.net/files/sp-client/sp-pages-assembly_en-us_dabadef8fce89e16a072e1ef7c166a4d.js?1709042329628 net::ERR_CONNECTION_RESET 200 (OK)
GET https://shell.cdn.office.net/shellux/en/shellstrings.52af792134b43bb66ac6fb020ec0b324.json net::ERR_HTTP2_PING_FAILED 200 (OK)

We don’t control the CDN profiles for res-1.cdn.office.net or res.cdn.office.net as they are the Microsoft ones, hence no changes from our end. Have there been any changes on the Microsoft end related to cdn?

The CDN we have set up are
*/USERPHOTO.ASPX
*/SITEASSETS
SITES/CONNECTDESIGNASSETS/SITETEMPLATES 
SITES/CONNECTDESIGNASSETS/APPROVEDPHOTOS
*/MASTERPAGE
*/STYLE LIBRARY       
*/CLIENTSIDEASSETS  

Examples of complete URL

https://tenant.sharepoint.com/SITES/CONNECTDESIGNASSETS/APPROVEDPHOTOS
https://tenant.sharepoint.com/SITES/CONNECTDESIGNASSETS/SITETEMPLATES


## SharePoint Backend

## Debugging 


## Networking
```
tracert tenant.sharepoint.com
```

sample output
```

```
1.	Page diagnostics tool
2.	Network tool


## References
https://martinliu.cn/2010/12/22/fiddler-timers/
https://connectivity.office.com/report/e8f2281a-df8e-4f05-93b2-949ba9b5958d

https://learn.microsoft.com/en-us/microsoft-365/enterprise/microsoft-365-ip-web-service?view=o365-worldwide
https://learn.microsoft.com/en-us/microsoft-365/enterprise/use-microsoft-365-cdn-with-spo?view=o365-worldwide

Enable the Microsoft 365 CDN | Microsoft Learn
