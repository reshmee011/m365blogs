---
title: 'Metalogix Import From FileShare To SharePoint - Extract Folder Name from FilePath using CSOM PnP'
date: Fri, 26 Aug 2016 21:05:05 +0000
draft: false
tags: ['CSOM', 'FilePath Extract Folder Name', 'Metalogix', 'Metalogix FileShare', 'PnP', 'SharePoint 2013']
---

I was evaluating the [Metalogix tool](http://www.metalogix.com/Products/Content-Matrix.aspx) for Migration from fileshare to SharePoint. The file share edition needs to be selected while installing the Metalogix content matrix tool. The good news is that it can fulfil the following requirements:

*   Migrate files recursively from within folders , i.e. flatten folder
*   Replace illegal characters with character specified
*   Handle duplicate by either overwriting file or not moving file
*   The CreatedBy, CreatedDate, ModifedBy and ModifiedDate file properties can be maintained by mapping these properties to SharePoint respective Out of the Box columns
*   Can apply custom filter on the following
    *   Created Before
    *   Created After
    *   Modified Before
    *   Modified After
    *   File Size Max

![MetalogixCustomFilters](https://reshmeeauckloo.files.wordpress.com/2016/08/metalogixcustomfilters.png)

*   Can migrate user permission as set on the folder/file from fileShare to SharePoint, however I could not find option how to map to SharePoint Groups.
*   Can migrate sub folders
*   Incremental migrate
*   Migration can be scheduled, in scenario where migration needs to be run out of office hours
*   Can extract custom properties on file using “Extracting MS Office File Properties”

I was stuck how to extract different level of folder names for nested folders. The folder names represent useful metadata which we wanted to migrate also. I managed to extract the top level folder using RegEx expression. I created a new column "Folder" from the "Items View "tab for the file share location. Right clicking on the view brings the option "Extract Content from Property" . ![MetalogixExtractContentFromProperty](https://reshmeeauckloo.files.wordpress.com/2016/08/metalogixextractcontentfromproperty.png) I used Regex ** \[^\\\\\]+(?=\\\\\[^\\\\\]+$)** on the Metalogix \[Source Url\] column and extracted the result to column \[Folder\] created above. ![MetalogixExtractFolderUsingRegex](https://reshmeeauckloo.files.wordpress.com/2016/08/metalogixextractfolderusingregex.png) After running the extraction, the column Folder is correctly populated with the folder name in which the file resides. ![MetalogixExtractFolderName](https://reshmeeauckloo.files.wordpress.com/2016/08/metalogixextractfoldername.png) I was trying to figure out how to retrieve various folder levels from a path and map them to different columns. Let's say the folder structure is \\Server\\First Level Folder\\Second Level Folder\\Third Level Folder\\My File.extension From the above method I managed to retrieve only \[Third Level Folder\] name. If I want to retrieve \[Second Level Folder\], then I have to use a different regex together **^(?:\[^\\\\\]\*\\\\){4}(\[^\\\\\]\*)** with the regex above ** \[^\\\\\]+(?=\\\\\[^\\\\\]+$)** _The number {4} can be updated to get different folder levels._ The regex can be tested from Windows PowerShell and correct result is returned by applying both regex expressions. ![TestRegexToRetrieveDifferentLevel](https://reshmeeauckloo.files.wordpress.com/2016/08/testregextoretrievedifferentlevel.png) I created two additional columns \[tmpLevel1\] and \[Level1\] from the file share location \[Items View\] tab. Using the  regex **^(?:\[^\\\\\]\*\\\\){4}(\[^\\\\\]\*),** I extracted the path into column \[tmpLevel1\] which is in the format " \\Server\\First Level Folder\\Second Level Folder\\Third Level Folder" ![MetalogixExtractFolderLevelUsingRegex](https://reshmeeauckloo.files.wordpress.com/2016/08/metalogixextractfolderlevelusingregex.png) I repeated the process but this time using **\[^\\\\\]+(?=\\\\\[^\\\\\]+$)** on column \[tmpLevel1\] to save result in column \[Level1\]. ![MetalogixExtractFolderLevelFinalUsingRegex](https://reshmeeauckloo.files.wordpress.com/2016/08/metalogixextractfolderlevelfinalusingregex.png) The final results is now I have the third level and second level folder name.![MetalogixExtractFolderLevelResultspng](https://reshmeeauckloo.files.wordpress.com/2016/08/metalogixextractfolderlevelresultspng.png) The issue however is that the steps need to be repeated for each new file share location making the deployment steps quite cumbersome. The other option is to keep the migration step simple mapping just Out of Box fields. ![MetalogixFieldMappings](https://reshmeeauckloo.files.wordpress.com/2016/08/metalogixfieldmappings.png) After files are migrated to SharePoint, the column \[MigrationSourceURL\] is created and populated by default in the library. ![MetalogixMigrationSourceURL](https://reshmeeauckloo.files.wordpress.com/2016/08/metalogixmigrationsourceurl.png) Thus running the PowerShell using CSOM script below after migration the columns Level0 and Level1 can be populated with the folder names. The value from column "MigrationSourceURL" is retrieved for all documents and split into level 0 and level 1. https://gist.github.com/reshmee011/dfe0b6fa3eb3d8f59e9d0415f66662d7 After running the script, the Level0 and Level1 columns are populated. ![DocumentsLevel01AfterPWCSOM](https://reshmeeauckloo.files.wordpress.com/2016/08/documentslevel01afterpwcsom.png)