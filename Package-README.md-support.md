Status: **Code Complete**

## Issue
The work for this feature was an intern project for summer 2017 and is based on discussions tracked here:
* **Support formatting in description [#2280](https://github.com/NuGet/NuGetGallery/issues/2280)**
* **Display extra information for packages hosted in GitHub [#2925](https://github.com/NuGet/NuGetGallery/issues/2925)**

## Problem
NuGet.org supports title and description metadata from the package [nuspec](https://docs.microsoft.com/en-us/nuget/schema/nuspec), but customers have expressed richer formatting support for the package details page.

## Who is the customer?
All package authors will be able to upload markdown documentation for their packages. This rich documentation benefits all customers who browse NuGet.org to discover new packages or to learn about updates to existing packages that they consume.

## Key Scenarios
The key scenarios we want to enable are:
* Enable authors to provide rich package documentation
* Enable authors to upload a README.md from a package repository

## Solution
The idea is to support display of a package README.md file. For the initial MVP, the idea is that the README.md can be uploaded by package authors to the NuGet.org site via text, url or file upload.

### Package Display View
-----
[[https://github.com/NuGet/Home/blob/dev/resources/ReadMeSupport/Display.PNG|Display README.md]]

### Package Upload / Edit View
-----
[[https://github.com/NuGet/Home/blob/dev/resources/ReadMeSupport/Upload.PNG|Upload README.md]]

Initial MVP restrictions include:
* Experience is limited to the NuGet.org site
* Upload must be done per package version
* No support for inline HTML in markdown
* Url upload is restricted to raw GitHub files

## Next Steps
The team will monitor use and feedback for the initial MVP on NuGet.org, and discuss the next steps required to extend support to clients. The ultimate goal is to support a README.md file packed into the actual package nupkg. Having README.md as part of the package allows for a more automated workflow, as well as extending support beyond just NuGet.org.

Note that the team may continue to discuss further GitHub integrations, but this is out of scope for the README.md feature.
