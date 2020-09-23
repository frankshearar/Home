* Status: **Reviewing**
* Authors: [Christopher Gill](https://github.com/chgill-msft) ([@chrisraygill](https://twitter.com/chrisraygill))

### Issue
The work for this feature and the discussion around the readme specific spec is tracked by **Nuspec - documentation/readme url [#6873](https://github.com/NuGet/Home/issues/6873)**

*****

### Description: What is it?

* Ability to easily include a README or similar in the package.
* Clearly display the package README on NuGet.org.
* Surface a link to the README in the Visual Studio NuGet UI.

### Problem: What problem is this solving?

Today authors struggle to communicate relevant and important information about their packages beyond a short description of their package on NuGet.org. Package consumers struggle to learn more about packages on NuGet.org and usually have to search elsewhere to even figure out what a package does.

Today, an author can include package documentation, but only post-upload. Due to the friction of this experience, fewer than 10% of packages use on NuGet.org use it (including very few top packages).

### Success: How do we know this is a real problem worth solving?

Customer interviews regarding the NuGet.org experience consistently mention the lack of documentation as a barrier to learning more about a package. Other package managers that allows embedded READMEs as a part of pack has significantly higher rates of documentation inclusion, and we frequently hear experience comparisons.

### Audience: Who are we building for?

All NuGet customers will benefit from this feature:
* Package authors will gain an easier workflow to attach relevant documentation with their packages and communicate important information about their packages to their customers.
* Package consumers will gain the ability to learn what a packages does, how to use the package, and other relevant information that the package author felt was important to communicate.

****

# Proposed design details

* The readme file will be embedded in the nupkg
* New nuspec property `<readme>README.md</readme>`
  * Will require a path relative to the package root to a readme file inside the package
  * Supported formats - md
  * the user will need to ensure the readme file is packed by adding a files element. E.g. `<file src="..\assets\readme.md" target="readme.md" />`
  * README.md files will **not** be automatically detected and included. Users are required to explicitly specify the file they want to use to avoid potential confusion and surprises.
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
* Very strech goal - Render the ReadMe inside the Visual Studio NuGet UI so package consumers can learn about package and reference usage without having to leave VS. This would work a lot like the VS Code extension manager.


### Project properties

![image](https://user-images.githubusercontent.com/15097183/93950924-e9161a80-fd12-11ea-9d0b-ae25c3bedb44.png)


```
    <PropertyGroup>
        <PackageReadmeFile>readme.md</PackageReadmeFile>
    </PropertyGroup>

    <ItemGroup>
        <None Include="..\assets\readme.md" Pack="true" PackagePath=""/>
    </ItemGroup>
```

### NuGet.org
* Upload from NuGet.org package preview - readme preview is rendered inline, similar to the license file.
* To avoid making the validations page unnecessarily long, the preview box should have a set max height and be scrollable  

![image](https://user-images.githubusercontent.com/15097183/93950678-26c67380-fd12-11ea-82ef-863a1430f414.png)

* NuGet.org package edit page will no longer allow you to edit the readme. Readme will be immutable and the user must push a new version if they want to make changes to the readme.
* It should be possible to have a URL to an on-page anchor to the readme section.
*  The first n lines of the readme should be visible by default. The readme length is > n, display "show more". Clicking on "show more" should have the same behavior as today.

![image](https://user-images.githubusercontent.com/15097183/89692400-6d762080-d8c0-11ea-8f37-7589bb83ca1b.png)

### 

### Validations
> + pack means `nuget pack`
> + push means `nuget push`
> + upload means https://www.nuget.org/packages/manage/upload 

* Size and other limitations for the md file that exists on NuGet.org still apply.
  * md file violates nuget.org size limits -> error on push, upload.
  * md contains unsupported syntax (Only core CommonMark features supported (no tables), and no inline HTML, injects “no follow” on all links) -> warn on pack. error on push, upload.
* readme is null -> warn on pack, push, upload. (users should remove that tag or add a meaningful readme.md file)

### FAQ

**Q: Will a warning or suggestion be surfaced if a README is not included in the nupkg?**

A: No. Private/internal package creation is a very common scenario with NuGet packages and does not require or necessarily benefit from README inclusion. Customers have generally indicated they _would_ be interested in an opt-in "strict pack validation" or package quality audit experience for when they are creating public packages.

**Q: Will a README be auto-detected and included if it exists at the package root?** 

A: No. Not all README files necessarily contain relevant package documentation today and many may not be in a state ready for NuGet.org consumption. We want to avoid surprising customers.

**Q: If there are images or GitHub Flavor Markdown specific formatting elements that are unsupported for NuGet.org in the included README file, will the package upload fail or succeed with warnings?**

A: The package upload will succeed with warnings. Customer feedback has indicated that they would rather upload a README file with missing images or improperly formatted elements than delay a planned package publish or update. A README is not "essential" content and will likely still be helpful even with some errors.


***