---
title: "Sharepoint Office Template Dotx"
date: 2024-05-14T00:39:57+01:00
draft: true
---

The issue you’re facing is an expected behavior because when you open a ".dotx" file in SharePoint, it opens the template for editing rather than creating a new ".docx" file from the template like it happens when we open it from Windows File Explorer.
While it is not possible to open a "*.dotx" template directly from SharePoint as a "*.docx" file, you can utilize the below steps as a workaround.
1.	Navigate to Site Contents -> Site Settings from the respective SharePoint site.
2.	Click on "Site content types" under "Web Designer Galleries".
3.	Create a new content type and click on it to open the same.
4.	In the next page, click on "Advance settings" -> Upload a new document template.
5.	Click on "Save".
6.	Now when the users click on "New" icon, they can select "Document" which you created/uploaded.
7.	Users will see a prompt that will say "The document you are trying to open is a template. Would you like to open and edit as a standard document".
8.	Users should click on "Yes" and click on "Open in Desktop App".
9.	This will now open the template you created/uploaded with name "Document1" in "*.docx" format.



Business Impact Statement (BIS) 
  
Please compose a "Business Impact Statement" (BIS) by answering the questions below in as much detail as possible. Try to put accurate numerical values for users being affected and loss of business, etc.

Please describe your organization:

Issue description: When attempting to open “.dotx” templates from SharePoint using the “Open in Desktop App” feature, the documents fail to open as “.docx” document. This issue prevents users from utilizing these templates directly from SharePoint in their desktop applications, impacting productivity.

Business Impact: 
  
1.	Which of the Microsoft 365 Apps channels is this fix needed in?
SharePoint

2.	Is the issue:
A. Affecting an existing deployment? A
B. Blocking from releasing a new deployment?
C. Blocking from deploying Microsoft Office 365 Apps for the first time?

3.	How are you using the product in question and what functionality are you attempting to provide to the business?

Unable to open a template in SharePoint – reverts to .dotx file extension

4.	How is the problem keeping you from performing specific business functions?

Have to use workaround which is to download template and save to file explorer and open from there

5.	If the problem is not resolved, how will it affect your business?
Example answer: We will be unable to use this add-in for the sales summary reporting, which will significantly impact the add-in's worth, and may require us to reconsider using Excel for this purpose.

It would not be ideal if issue is not resolved as all templates are saved in SharePoint and to keep downloading templates and saving to file explorer will waste our time.

6.	How many users are affected by this issue?
Example answer: 200 of our 10,000 users currently cannot use this add-in with Excel.
 
Less than 10 users are affected

7.	How frequently are those users affected by this issue?
Example answer: Daily/Weekly/Monthly, 25% of the time, or inconsistent.

each time a template is used the user is affected
templates are used on a daily basis

8.	What is the financial impact of this issue?
Example answer: $30,000 for the sales unit on add-in development, and $2,000 per month in lost productivity.

 no financial impact, however time is wasted by downloading the templates
It should be as simple as opening from sharepoint without the need of downloading

9.	What are the proposed workarounds and why are they not acceptable?

workaround is to download template and save to file explorer or sync sharepoint site via onedrive file explorer.

Workaround is fine, however, would prefer the templates to open directly in sharepoint as they are saved there currently.
