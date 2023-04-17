---
title: 'Updating Resources Custom Field values using CSOM, Project Online, #Project Server 2013'
date: Wed, 18 Nov 2015 18:05:30 +0000
draft: false
tags: ['C#', 'CSOM', 'Project Online', 'Project Server 2013', 'SharePoint 2013']
---


#### 
[anil]( "anilcareer.b.tech@gmail.com") - <time datetime="2016-05-27 19:11:56">May 5, 2016</time>

You can update custom field using csom. Follow the link below http://projectservercode.com/tag/how-to-update-custom-field-using-csom/
<hr />
#### 
[Raghava Reddy](https://www.epmsolutions.com/ "raghava@epmsolutions.com") - <time datetime="2018-05-28 04:55:34">May 1, 2018</time>

Hi Anil. i am getting below error, can you please suggest me if there are any other way to fix it. projContext.ExecuteQuery(); An unhandled exception of type 'Microsoft.SharePoint.Client.ServerException' occurred in Microsoft.SharePoint.Client.Runtime.dll Additional information: PJClientCallableException: CustomFieldRequiredValueNotProvided Below is my code. static string UserName = "raghava@hosting.com"; static string Passwords = "password"; static void Main(string\[\] args) { string pwaPath = "https://epmsolutions.sharepoint.com/sites/PWA3/"; using (PRCLNT.ProjectContext projContext = new PRCLNT.ProjectContext(pwaPath))//PWA Url { SecureString passWord4 = new SecureString(); foreach (char c in Passwords.ToCharArray()) passWord4.AppendChar(c); projContext.Credentials = new SharePointOnlineCredentials(UserName, passWord4); Guid resUID = new Guid("664f2fd5-6e21-e611-80db-00155d50d126"); string customFieldName = "test res"; projContext.Load(projContext.EnterpriseResources); projContext.Load(projContext.CustomFields); projContext.ExecuteQuery(); Console.WriteLine("\\nResource ID : Resource name "); int numResInCollection = projContext.EnterpriseResources.Count(); var usrs = projContext.Web.SiteUsers; if (numResInCollection > 0) { projContext.Load(projContext.EnterpriseResources.GetByGuid(resUID)); //projContext.Load(projContext.EnterpriseResources.GetByGuid(resUID)); projContext.Load(projContext.EntityTypes.ResourceEntity); projContext.ExecuteQuery(); var entRes2Edit = projContext.EnterpriseResources.GetByGuid(resUID); var userCustomFields = entRes2Edit.CustomFields; Guid ResourceEntityUID = projContext.EntityTypes.ResourceEntity.ID; var customfield = projContext.CustomFields.Where(x => x.Name == customFieldName); //entRes2Edit\[customfield.First().InternalName\] ="333"; //entRes2Edit\[customFieldName\] = "333"; entRes2Edit.FieldValues.Add(customfield.First().InternalName,"333"); Console.WriteLine("\\nEditing resource : GUID : Can Level"); Console.WriteLine("\\n{0} : {1} : {2}", entRes2Edit.Name, entRes2Edit.Id.ToString(),entRes2Edit.CanLevel.ToString()); entRes2Edit.CanLevel = !entRes2Edit.CanLevel; projContext.EnterpriseResources.Update(); projContext.ExecuteQuery(); // ERRROOORRR projContext.Load(projContext.EnterpriseResources.GetByGuid(resUID)); projContext.ExecuteQuery(); entRes2Edit = projContext.EnterpriseResources.GetByGuid(resUID); Console.WriteLine("\\n\\nChanged resource : GUID : Can Level"); Console.WriteLine("\\n{0} : {1} : {2}", entRes2Edit.Name, entRes2Edit.Id.ToString(), entRes2Edit.CanLevel.ToString()); } Console.Write("\\nPress any key to exit: "); Console.ReadKey(false); } }
<hr />
#### 
[Raghava Reddy](https://www.epmsolutions.com/ "raghava@epmsolutions.com") - <time datetime="2018-05-24 14:40:10">May 4, 2018</time>

Hi Anil, Thanks, i am not able to open link which suggested by you. i tried with below sample but getting same issue. //entRes2Edit\[customfield.First().InternalName\] ="333"; //entRes2Edit\[customFieldName\] = "333"; entRes2Edit.FieldValues.Add(customfield.First().InternalName,"333"); can you please guide me to fix this issue. if you can please send code from suggested link.
<hr />
#### 
[Raghava Reddy](https://www.epmsolutions.com/ "raghava@epmsolutions.com") - <time datetime="2018-05-23 13:08:09">May 3, 2018</time>

Hi i am getting below, can you please tell me how can we fix it. projContext.ExecuteQuery(); An unhandled exception of type 'Microsoft.SharePoint.Client.ServerException' occurred in Microsoft.SharePoint.Client.Runtime.dll Additional information: PJClientCallableException: CustomFieldRequiredValueNotProvided thanks.
<hr />
#### 
[Raghava Reddy](https://www.epmsolutions.com/ "raghava@epmsolutions.com") - <time datetime="2018-05-29 07:09:45">May 2, 2018</time>

Hi Anil, now i fix the issue, i was getting that issue because, resource was have a required field with no data. so i changed required field as not required, then code our code is able to update resource custom fields. Thanks for the help and support.
<hr />
#### 
[greg winterhalter]( "gwinterhalter@epmsolutions.com") - <time datetime="2018-06-04 00:55:54">Jun 1, 2018</time>

Are you interested in some Project Online CSOM consulting.
<hr />
