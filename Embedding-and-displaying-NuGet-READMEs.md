* Status: **Reviewing**
* Author(s): [Karan Nandwani](https://github.com/karann-msft) ([@karann9](https://twitter.com/karann9)) and [Christopher Gill](https://github.com/chgill-msft) ([@chrisraygill](https://twitter.com/chrisraygill))

## Issue
The work for this feature and the discussion around the readme specific spec is tracked by **Nuspec - documentation/readme url [#6873](https://github.com/NuGet/Home/issues/6873)**

*****

## Description: What is it?

* Ability to easily include a README or similar in the package.
* Clearly display the package README on NuGet.org.
* Surface a link to the README in the Visual Studio NuGet UI.

## Problem: What problem is this solving?

Today authors struggle to communicate relevant and important information about their packages beyond a short description of their package on NuGet.org. Package consumers struggle to learn more about packages on NuGet.org and usually have to search elsewhere to even figure out what a package does.

Today, an author can include package documentation, but only post-upload. Due to the friction of this experience, fewer than 10% of packages use on NuGet.org use it (including very few top packages).

## Success: How do we know this is a real problem worth solving?

Customer interviews regarding the NuGet.org experience consistently mention the lack of documentation as a barrier to learning more about a package. Other package managers that allows embedded READMEs as a part of pack has significantly higher rates of documentation inclusion, and we frequently hear experience comparisons.

## Audience: Who are we building for?

All NuGet customers will benefit from this feature:
* Package authors will gain an easier workflow to attach relevant documentation with their packages and communicate important information about their packages to their customers.
* Package consumers will gain the ability to learn what a packages does, how to use the package, and other relevant information that the package author felt was important to communicate.

****

# Proposed design details

* The readme file will be a part of the nupkg
* New nuspec property `<readme>readme.md</readme>`
  * Is a path relative to the package root to a readme file inside the package`
  * Supported formats - md
  * the user will need to ensure the readme file is packed by adding a files element. E.g. `<file src="..\assets\readme.md" target="readme.md" />`
* If `readme.md` is present at the folder/project root, and readme property is not present in the nuspec/project file, nuget pack should pack that file as the package readme.
* Browse from NuGet.org
  * During package ingestion, nuget.org will extract and validate the md, and update the package details page with the content.
  * Client will surface the readme URL served by the gallery
  * Clicking on the link will open the default browser and go to the package details page that contains readme (and that section will be expanded and scroll down so that the readme is at the top of the page)
  * We will only surface links to documentation for packages with embedded readmes. Legacy post-publish documentation that currently exists on nuget.org will not be linked in client due to low adoption.
![image](https://user-images.githubusercontent.com/15097183/89691998-4b2fd300-d8bf-11ea-83c1-6b4205d33229.png)
* Stretch goal - Installed packages/folder based feeds/fallback folder
  * Client will provide a link to open the readme file from the nupkg/global packages folder/extracted location
  * Clicking on the link will open the file in the default application associated with `.md` extension
  * Client will do the same validations and security checks as nuget.org before displaying md files.
* Very strech goal - Render the ReadMe inside the Visual Studio NuGet UI so package consumers can learn about package and reference usage without having to leave VS. This would work a lot like the VS Code extension manager. (Would require a massive redesign of the Visual Studio NuGet UI)


#### Project properties

![image](https://user-images.githubusercontent.com/15097183/86620008-72b61780-bf70-11ea-9c3a-465e7ceec1ad.png)


```
    <PropertyGroup>
        <PackageReadmeFile>readme.md</PackageReadmeFile>
    </PropertyGroup>

    <ItemGroup>
        <None Include="..\assets\readme.md" Pack="true" PackagePath=""/>
    </ItemGroup>
```

#### NuGet.org
* Upload from NuGet.org package preview - readme preview is rendered inline, similar to the license file.
* To avoid making the validations page unnecessarily long, the preview box should have a set max height and be scrollable  

![image](https://user-images.githubusercontent.com/16904420/52312144-57e80980-295e-11e9-95cf-cc33ac1261b3.png)

* NuGet.org package edit page will no longer allow you to edit the readme. Readme will be immutable and the user must push a new version if they want to make changes to the readme.
* Saying "readme" feels redundant. Also, readme itself tends to have headers. These headers should start from h2 instead of showing the word "Documentation" at h2.
* It should be possible to have a URL to an on-page anchor to the readme section.
*  The first n lines of the readme should be visible by default. The readme length is > n, display "show more". Clicking on "show more" should have the same behavior as today.

![image](https://user-images.githubusercontent.com/15097183/89692400-6d762080-d8c0-11ea-8f37-7589bb83ca1b.png)

#### Validations
> + pack means `nuget pack`
> + push means `nuget push`
> + upload means https://www.nuget.org/packages/manage/upload 

* Size and other limitations for the md file that exists on NuGet.org still apply.
  * md file violates nuget.org size limits -> error on push, upload.
  * md contains unsupported syntax (Only core CommonMark features supported (no tables), and no inline HTML, injects “no follow” on all links) -> warn on pack. error on push, upload.
* readme is null -> warn on pack, push, upload. (users should remove that tag or add a meaningful readme.md file)

#### Open questions
* If a README contains formatting or images that is unsupported by NuGet.org, should we:
   * Reject the package and provide a strong error experience.
   * Accept the package, surface warnings, and don't show the README.
   * Accept the package, provide warnings, but still show the README with potentially missing or unformatted content.
* Should we auto-detect and include a README if it exists at the package root? (Less effort on the package author's side, but they might be surprised if this was unintended.
* Should we push for greater adoption of this feature by:
   * Warning on pack if no README is included.
   * Printing non-warning "soft suggestions" to include a README.
   * Rely on evangelizing the feature or include it as part of a quality metric in the future.

***
[Back to parent spec](https://github.com/NuGet/Home/wiki/Packaging-Icon,-License-and-Documentation-within-the-nupkg)

[Packaging Icon within the nupkg](https://github.com/NuGet/Home/wiki/Packaging-Icon-within-the-nupkg)

[Packaging License within the nupkg](https://github.com/NuGet/Home/wiki/Packaging-License-within-the-nupkg)