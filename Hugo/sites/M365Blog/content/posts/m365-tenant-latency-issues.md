---
title: "M365 Tenant Latency Issues"
date: 2024-02-29T10:56:54Z
omit_header_text: true
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


https://publiccdn.sharepointonline.com/{tenantname}/sites/appcatalog/ClientSideAssets/81faa992-a943-4109-8fa0-c4d89d3a2319/PromoteNewsCommandSetStrings_en-gb_0ce05a97be92588f79473ea846c204cb.js

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
3. win mtr  tool


Step 1: Start the traces -Open the command prompt (as admin) and run the below command.
Netsh trace start scenario=NetConnection,wfp-ipsec capture=yes report=yes filemode=circular overwrite=yes maxsize=1024 persistent=yes tracefile=c:\%computername%_nettrace.etl
 
Step 2: Reproduce the issue
 
Step 3: Stop the traces -From command prompt
Netsh trace stop
 
Step 4:  Upload the traces :

We did see delays in in this scenario and we could see it starting from the VPN and it could be an intermittent hop which is causing the delay.


We also checked tracert to res-1.cdn.office.com and we could see the delay on intermediate node as well which was ISP in this case.

SPO CDN investigation results:

Looking at attached logs it looks like there are delays for request that are going to https://shell.cdn.office.net and https://res-1.cdn.office.net . https://shell.cdn.office.net is not part of 1CD so this issue shouldn't be exclusive for 1CDN. 

Here is one example for  https://res-1.cdn.office.net  request:

 

Server Timing is 40ms and the rest is client network. 70% if a delay are Queueing and Stalled. 

According to Network features reference - Microsoft Edge Developer documentation | Microsoft Learn
•	Queueing. The browser queues requests when any of the following are true
o	There are higher priority requests.
o	There are already six TCP connections open for this origin, which is the limit. Applies to HTTP/1.0 and HTTP/1.1 only.
o	The browser is briefly allocating space in the disk cache.
•	Stalled. The request could be stalled for any of the reasons described in Queueing.

The TCP and the TLS handshake is completing b/w the client and the server. 
After the TLS handshake I could see multiple dup-acks and retransmits being communicated b/w the client and the server. 
Dup-acks are coming from our client and the fast retransmission is from the server. 
At last, the server has sent a TCP reset aborting the connection. 
Team, as per the traces, I would say that the delay is caused due to the dup acks and the retransmits. It can be due to the 
network congestion, packet loss, or errors on the intermediate devices involved in the communication. 


## References
https://martinliu.cn/2010/12/22/fiddler-timers/
https://connectivity.office.com/report/e8f2281a-df8e-4f05-93b2-949ba9b5958d
https://learn.microsoft.com/en-us/microsoft-365/enterprise/use-microsoft-365-cdn-with-spo?view=o365-worldwide
https://learn.microsoft.com/en-gb/microsoft-edge/devtools-guide-chromium/network/reference
https://learn.microsoft.com/en-us/microsoft-365/enterprise/microsoft-365-ip-web-service?view=o365-worldwide
https://learn.microsoft.com/en-us/microsoft-365/enterprise/use-microsoft-365-cdn-with-spo?view=o365-worldwide

Enable the Microsoft 365 CDN | Microsoft Learn
