Status: **Implemented** on NuGet.org

## Issue
The work for this feature was an **intern project** for summer 2017 and is based on discussions tracked here:
* **Support formatting in description [#2280](https://github.com/NuGet/NuGetGallery/issues/2280)**. Please use this issue to discuss or provide feedback on the feature.
* **Display extra information for packages hosted in GitHub [#2925](https://github.com/NuGet/NuGetGallery/issues/2925)**

## Problem
NuGet.org supports title and description metadata from the package [nuspec](https://docs.microsoft.com/en-us/nuget/schema/nuspec), but customers have expressed richer formatting support for the package details page.

## Who is the customer?
All package authors will be able to upload markdown documentation for their packages. This rich documentation benefits all customers who browse NuGet.org to discover new packages or to learn about updates to existing packages that they consume.

## Key Scenarios
The key scenarios we want to enable are:
* Enable authors to provide rich package documentation
* Enable authors to upload a documentation markdown file from a package repository

## Solution
The idea is to support rich package documentation via markdown content. For the initial MVP, the idea is that the markdown can be can be uploaded by package authors to the NuGet.org site via text, url or file upload.

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
The team will monitor use and feedback for the initial MVP on NuGet.org, and discuss the next steps required to extend support to clients. **The ultimate goal is to support a markdown file packed into the actual package nupkg**. Having a markdown file as part of the package allows for a more automated workflow, as well as extending support beyond just NuGet.org.

Note that the team may continue to discuss further GitHub integrations, but this is out of scope for this feature.
