---
title: "How to Hide the 'See All' Button in the Highlighted Content Web Part using PnP PowerShell"
date: 2023-11-01T15:14:29Z
tags: ["Modern SharePoint Page","WebPart","PowerShell", "Set-PnPPageWebPart"]
featured_image: '/posts/images/PowerShell_HighlightWebPart_HideSeeAll/HighlightWebpart.png'
omit_header_text: true
draft: false
---

# How to Hide the "See All" Button in the Highlighted Content Web Part using PnP PowerShell

Recently, I encountered an issue with the "Show Title and Commands" toggle in the out-of-the-box Highlighted Content web part. It stopped working on both my development and customer tenant. I've raised the issue on the [Microsoft Forum](https://answers.microsoft.com/en-us/msoffice/forum/all/toggle-show-title-and-commands-for-highlighted/f74fd668-2fac-45e3-a171-1563494c01c1) and also opened a case with Microsoft to investigate the backend. 

![ToggleOff](../images/PowerShell_HighlightWebPart_HideSeeAll/HighlightWebpart.png)

While awaiting a resolution from Microsoft, I decided to find a workaround. That's when the PnP PowerShell cmdlet `Set-PnPPageWebPart` came to the rescue. The solution involves updating the **PropertiesJson** property of the web part.

For the Highlighted Content web part, here's an example of the Json property:

```json
{"query":{"contentLocation":1,"contentTypes":[1],"sortType":1,"filters":[{"filterType":1,"value":""}]},"templateId":1,"maxItemsPerPage":8,"hideWebPartWhenEmpty":false,"sites":[],"queryMode":"Basic","isTitleEnabled":true,"layoutId":"Card","dataProviderId":"Search","displayMaps":{"1":{"headingText":{"sources":["SiteTitle"]},"headingUrl":{"sources":["SPWebUrl"]},"title":{"sources":["UserName","Title"]},"personImageUrl":{"sources":["ProfileImageSrc"]},"name":{"sources":["Name"]},"initials":{"sources":["Initials"]},"itemUrl":{"sources":["WebPath"]},"activity":{"sources":["ModifiedDate"]},"previewUrl":{"sources":["PreviewUrl","PictureThumbnailURL"]},"iconUrl":{"sources":["IconUrl"]},"accentColor":{"sources":["AccentColor"]},"cardType":{"sources":["CardType"]},"tipActionLabel":{"sources":["TipActionLabel"]},"tipActionButtonIcon":{"sources":["TipActionButtonIcon"]},"className":{"sources":["ClassName"]},"telemetryProperties":{"sources":["TelemetryProperties"]},"imageOverlapText":{"sources":["ImageOverlapText"]},"imageOverlapTextAriaLabel":{"sources":["ImageOverlapTextAriaLabel"]},"spWebUrl":{"sources":["SPWebUrl"]},"fileType":{"sources":["FileType"]},"UniqueID":{"sources":["UniqueID"]},"iframeUrl":{"sources":["iframeUrl"]},"isVideoItem":{"sources":["isVideoItem"]}},"2":{"column1":{"heading":"Filetype","sources":["FileType"],"width":34},"column2":{"heading":"Title","sources":["Title"],"linkUrls":["WebPath"],"width":250},"column3":{"heading":"Modified","sources":["ModifiedDate"],"width":100},"column4":{"heading":"Modified By","sources":["Name"],"width":150}},"3":{"id":{"sources":["UniqueID"]},"edit":{"sources":["edit"]},"DefaultEncodingURL":{"sources":["DefaultEncodingURL"]},"FileExtension":{"sources":["FileExtension"]},"FileType":{"sources":["FileType"]},"imageOverlapText":{"sources":["ImageOverlapText"]},"imageOverlapTextAriaLabel":{"sources":["ImageOverlapTextAriaLabel"]},"Path":{"sources":["Path"]},"PictureThumbnailURL":{"sources":["PictureThumbnailURL"]},"PreviewUrl":{"sources":["PreviewUrl"]},"SiteID":{"sources":["SiteID"]},"SiteTitle":{"sources":["SiteTitle"]},"Title":{"sources":["Title"]},"UniqueID":{"sources":["UniqueID"]},"WebId":{"sources":["WebId"]},"WebPath":{"sources":["WebPath"]},"spWebUrl":{"sources":["SPWebUrl"]},"fileType":{"sources":["FileType"]},"iframeUrl":{"sources":["iframeUrl"]},"isVideoItem":{"sources":["isVideoItem"]}},"4":{"headingText":{"sources":["SiteTitle"]},"headingUrl":{"sources":["SPWebUrl"]},"title":{"sources":["UserName","Title"]},"personImageUrl":{"sources":["ProfileImageSrc"]},"name":{"sources":["Name"]},"initials":{"sources":["Initials"]},"itemUrl":{"sources":["WebPath"]},"activity":{"sources":["ModifiedDate"]},"previewUrl":{"sources":["PreviewUrl","PictureThumbnailURL"]},"iconUrl":{"sources":["IconUrl"]},"accentColor":{"sources":["AccentColor"]},"cardType":{"sources":["CardType"]},"tipActionLabel":{"sources":["TipActionLabel"]},"tipActionButtonIcon":{"sources":["TipActionButtonIcon"]},"className":{"sources":["ClassName"]},"telemetryProperties":{"sources":["TelemetryProperties"]},"imageOverlapText":{"sources":["ImageOverlapText"]},"imageOverlapTextAriaLabel":{"sources":["ImageOverlapTextAriaLabel"]},"spWebUrl":{"sources":["SPWebUrl"]},"fileType":{"sources":["FileType"]},"UniqueID":{"sources":["UniqueID"]},"iframeUrl":{"sources":["iframeUrl"]},"isVideoItem":{"sources":["isVideoItem"]}},"6":{"headingText":{"sources":["SiteTitle"]},"headingUrl":{"sources":["SPWebUrl"]},"title":{"sources":["UserName","Title"]},"personImageUrl":{"sources":["ProfileImageSrc"]},"name":{"sources":["Name"]},"initials":{"sources":["Initials"]},"itemUrl":{"sources":["WebPath"]},"activity":{"sources":["ModifiedDate"]},"previewUrl":{"sources":["PreviewUrl","PictureThumbnailURL"]},"iconUrl":{"sources":["IconUrl"]},"accentColor":{"sources":["AccentColor"]},"cardType":{"sources":["CardType"]},"tipActionLabel":{"sources":["TipActionLabel"]},"tipActionButtonIcon":{"sources":["TipActionButtonIcon"]},"className":{"sources":["ClassName"]},"telemetryProperties":{"sources":["TelemetryProperties"]},"imageOverlapText":{"sources":["ImageOverlapText"]},"imageOverlapTextAriaLabel":{"sources":["ImageOverlapTextAriaLabel"]},"spWebUrl":{"sources":["SPWebUrl"]},"fileType":{"sources":["FileType"]},"UniqueID":{"sources":["UniqueID"]},"iframeUrl":{"sources":["iframeUrl"]},"isVideoItem":{"sources":["isVideoItem"]}}},"webId":"f9d0ac17-0a85-4b1b-937d-e106eb50af40","siteId":"0e969b42-9107-4e0a-a16d-77ba2ea54ec3"}
```

I initially tried to use ConvertFrom-Json to handle the properties, but encountered an error due to keys with different casing.
```PowerShell
$webpart.PropertiesJson|convertfrom-json -depth 10

ConvertFrom-Json: Cannot convert the JSON string because it contains keys with different casing. Please use the -AsHashTable switch instead. The key that was
attempted to be added to the existing key 'FileType' was 'fileType'.
```

To work around this, I used a string replace function to change the property "isTitleEnabled" to false. This successfully hid the "See All" button while keeping the title visible.
 
```powershell
# Connect to your SharePoint site
Connect-PnPOnline -Url "https://contoso.sharepoint.com/sites/Project" -Interactive

# Specify the page URL
$pageUrl = "ProjectHome.aspx"

# Get the page and its web parts$
$page = Get-PnPClientSidePage -Identity $pageUrl
$webParts = $page.Controls | Where-Object { $_.Title -eq 'Highlighted content' } 

# Loop through each web part
foreach ($webPart in $webParts) {
        # Update isTitleEnabled property within PropertiesJson
        $jsonUp = $webpart.PropertiesJson.Replace('"isTitleEnabled":true','"isTitleEnabled":false') 
        
        Set-PnPPageWebPart -Page $pageUrl -Identity $webPart.InstanceId -PropertiesJson $jsonUp
    }

# Disconnect from the SharePoint site
Disconnect-PnPOnline
```

The "See All" button successfully hidden.

![ToggleOff](../images/PowerShell_HighlightWebPart_HideSeeAll/ToggleOff.png)

## Conclusion

In this post, we explored a workaround for an issue with the "Show Title and Commands" toggle in the Highlighted Content web part in SharePoint. While awaiting a resolution from Microsoft, we leveraged the power of PnP PowerShell and the Set-PnPPageWebPart cmdlet to update the PropertiesJson property of the web part.

## References

[Toggle "Show Title and Commands" for Highlighted webpart in SharePoint not working](https://answers.microsoft.com/en-us/msoffice/forum/all/toggle-show-title-and-commands-for-highlighted/f74fd668-2fac-45e3-a171-1563494c01c1)