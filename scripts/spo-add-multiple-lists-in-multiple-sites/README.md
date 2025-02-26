---
plugin: add-to-gallery
---

# Add multiple lists in multiple sites

## Summary

Create multiple lists in multiple sites and also to map the content type

# [CLI for Microsoft 365 with PowerShell](#tab/cli-m365-ps)
```powershell
<#
.SYNOPSIS
    Create multiple lists in multiple sites.
.DESCRIPTION
    Create multiple lists in multiple sites and also to map the content type.
.EXAMPLE
    PS C:\> Add-ListsAndMapContentTypes -SiteUrls "/Sites/Site1", "/Sites/Site2" -Lists "List1", "List2"
.INPUTS
    Inputs (if any)
.OUTPUTS
    Output (if any)
.NOTES
    Below are the 2 global variables that have to be updated
    $weburl = "https://tenantname.sharepoint.com"
    $ContentTypeId = '<content type id>'
#>
$weburl = "https://tenantname.sharepoint.com"
$ContentTypeId = '<content type id>'

function Add-ListsAndMapContentTypes (
  [Parameter(Mandatory = $true)][string[]] $SiteUrls,
  [Parameter(Mandatory = $true)][string[]] $Lists 
) {
  $SiteUrls | ForEach-Object {
    [string]$FinalSiteUrl = $weburl + $_.ToString()
    Write-Output "Started creating lists for '$FinalSiteUrl'"
    Write-Output ""
    $Lists | ForEach-Object {
      [string]$listTitle = $_.ToString()
      Write-Output "Checking and creating list '$listTitle'"
      $list = m365 spo list get -u $FinalSiteUrl -t $listTitle -o 'json' | ConvertFrom-Json
      if ($null -eq $list) {
        m365 spo list add -t $listTitle --baseTemplate DocumentLibrary -u $FinalSiteUrl --contentTypesEnabled true --enableVersioning true --listExperienceOptions 1 --onQuickLaunch false 
        $list = m365 spo list get -t $listTitle -u $FinalSiteUrl --properties "Title,Id" --output 'json' | ConvertFrom-Json
        m365 spo list contenttype add -l $list.Id -u $FinalSiteUrl -c $ContentTypeId --output 'json' | ConvertFrom-Json
        $listContentType = m365 spo list contenttype list -l $list.Id -u $FinalSiteUrl --output 'json' | ConvertFrom-Json
        m365 spo list contenttype default set -l $list.Id -u $FinalSiteUrl -c $listContentType.StringId[2] --output 'json' | ConvertFrom-Json
        Write-Output "Successfully created list '$listTitle'"
      }
      else {
        Write-Output "List '$listTitle' already exists"
      }
      Write-Output "----------------------"
    }
    Write-Output ""
  }
}

Write-Host "Ensure logged in"
$m365Status = m365 status
if ($m365Status -eq "Logged Out") {
  Write-Host "Logging in the User!"
  m365 login --authType browser
}
```
[!INCLUDE [More about CLI for Microsoft 365](../../docfx/includes/MORE-CLIM365.md)]


## Source Credit

Sample first appeared on [Add multiple lists in multiple sites | CLI for Microsoft 365](https://pnp.github.io/cli-microsoft365/sample-scripts/spo/add-multiple-lists-in-multiple-sites/)

## Contributors

| Author(s) |
|-----------|
| Sudharsan Kesavanarayanan |


[!INCLUDE [DISCLAIMER](../../docfx/includes/DISCLAIMER.md)]
<img src="https://pnptelemetry.azurewebsites.net/script-samples/scripts/spo-add-multiple-lists-in-multiple-sites" aria-hidden="true" />
