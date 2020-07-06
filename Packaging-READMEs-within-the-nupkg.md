* Status: **Reviewing**
* Author(s): [Karan Nandwani](https://github.com/karann-msft) ([@karann9](https://twitter.com/karann9)) and [Christopher Gill](https://github.com/chgill-msft) ([@chrisraygill](https://twitter.com/chrisraygill))

### Issue
The work for this feature and the discussion around the readme specific spec is tracked by **Nuspec - documentation/readme url [#6873](https://github.com/NuGet/Home/issues/6873)**

### Client

****

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
  * We will only surface links to documentation for packages with embedded readmes. Legacy post-publish documentation that currently exists on nuget.org will not be linked in client due to low ROI (there are very few packages that use the existing documentation flow).
![image](https://user-images.githubusercontent.com/16904420/52244182-3f5ded80-2891-11e9-875c-beddcaf49e2b.png)
* Stretch goal - Installed packages/folder based feeds/fallback folder
  * Client will provide a link to open the readme file from the nupkg/global packages folder/extracted location
  * Clicking on the link will open the file in the default application associated with `.md` extension
  * Client will do the same validations and security checks as nuget.org before displaying md files. **This sounds like a dry-run push/publish**
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

![image](https://user-images.githubusercontent.com/16904420/52377004-7b6f8a80-2a18-11e9-897a-6d6b99bd6b90.png)

#### Admin flow - Tentative based on frequency of relevant copyright scenario
* Similar to icon, NuGet.org admin view to remove the package documentation and block it from being displayed on NuGet.org or in VS during browse from NuGet.org.
![image](https://user-images.githubusercontent.com/16904420/52311447-d0010000-295b-11e9-89cc-b5142caaf672.png)

#### Validations
> + pack means `nuget pack`
> + push means `nuget push`
> + upload means https://www.nuget.org/packages/manage/upload 

* Size and other limitations for the md file that exists on NuGet.org still apply.
  * md file violates nuget.org size limits -> error on push, upload.
  * md contains unsupported syntax (Only core CommonMark features supported (no tables), and no inline HTML, injects “no follow” on all links) -> warn on pack. error on push, upload.
* readme is null -> warn on pack, push, upload. (users should remove that tag or add a meaningful readme.md file)

#### Open questions
* Should we warn on pack and upload if there is no readme attached?
  * Pro: Higher rate of readme inclusion/adoption
  * Con: More friction for beginners getting started?
  * **Current proposal: This is should come after some months of monitoring adoption. We don't want to be too "strict" with this feature until we understand if there are any customer friction points associated with it.**

* Should we make a readme a requirement for prefix reservation?
  * **Current proposal: Same as above.**

* Can we make ReadMe links easily accessible or clickable from the dotnet CLI and/or nuget.exe?
* Can we populate the "readme" field in the package properties if we detect a ReadMe.md at the package root even if the property isn't specified (like we will during pack).
* Is there a way to preview how a markdown file will render on nuget.org without the manual upload process?
* Is it possible for us to support GitHub's markdown flavor to ensure we don't have mismatching limitations?

***
[Back to parent spec](https://github.com/NuGet/Home/wiki/Packaging-Icon,-License-and-Documentation-within-the-nupkg)

[Packaging Icon within the nupkg](https://github.com/NuGet/Home/wiki/Packaging-Icon-within-the-nupkg)

[Packaging License within the nupkg](https://github.com/NuGet/Home/wiki/Packaging-License-within-the-nupkg)