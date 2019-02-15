* Status: **Reviewing**
* Author(s): [Karan Nandwani](https://github.com/karann-msft) ([@karann9](https://twitter.com/karann9))

### Issue
The work for this feature and the discussion around the readme specific spec is tracked by **Nuspec - documentation/readme url [#6873](https://github.com/NuGet/Home/issues/6873)**

### Readme

* The readme file will be a part of the nupkg
* New nuspec property `<readme>readme.md</readme>`
  * Is a path relative to the package root to a readme file inside the package`
  * Supported formats - md
  * the user will need to ensure the readme file is packed by adding a files element. E.g. `<file src="..\assets\readme.md" target="readme.md" />`
* If `readme.md` is present at the folder/project root, and readme property is not present in the nuspec/project file, nuget pack should pack that file as the package readme.
* Browse from NuGet.org
  * During package ingestion, nuget.org will extract and validate the md, and update the package details page with the content.
  * Client will surface the readme URL served by the gallery
  * Clicking on the link will open the default browser and go to the package details page that contains readme (and that section will be expanded).
* Stretch goal - Installed packages/folder based feeds/fallback folder
  * Client will provide a link to open the readme file from the nupkg/global packages folder/extracted location
  * Clicking on the link will open the file in the default application associated with `.md`extension
  * Client will do the same validations and security checks as nuget.org before displaying md files.
  ![image](https://user-images.githubusercontent.com/16904420/52244182-3f5ded80-2891-11e9-875c-beddcaf49e2b.png)

#### Project properties

![image](https://user-images.githubusercontent.com/16904420/52376505-254e1780-2a17-11e9-9bc8-a85258490c59.png)


```
    <PropertyGroup>
        <PackageReadmeFile>readme.md</PackageReadmeFile>
    </PropertyGroup>

    <ItemGroup>
        <None Include="..\assets\readme.md" Pack="true" PackagePath=""/>
    </ItemGroup>
```

#### NuGet.org
* Upload from NuGet.org package preview - readme preview is rendered inline, similar to the license file. (all the extra whitespace in the mock below is unintentional)
  ![image](https://user-images.githubusercontent.com/16904420/52312144-57e80980-295e-11e9-95cf-cc33ac1261b3.png)
* NuGet.org package edit page will no longer allow you to edit the readme. Readme will be immutable and the user must push a new version if they want to make changes to the readme.
* Saying "Documentation" feels redundant. Also, readme itself tends to have headers. These headers should start from h2 instead of showing the word "Documentation" at h2.
* It should be possible to have a URL to an on-page anchor to the readme section.
*  The first n lines of the readme should be visible by default. The readme length is > n, display "show more". Clicking on "show more" should have the same behavior as today.
![image](https://user-images.githubusercontent.com/16904420/52377004-7b6f8a80-2a18-11e9-897a-6d6b99bd6b90.png)

#### Admin Flow
* Similar to icon, NuGet.org admin view to remove the package documentation and block it from being displayed on NuGet.org or in VS during browse from NuGet.org.
![image](https://user-images.githubusercontent.com/16904420/52311447-d0010000-295b-11e9-89cc-b5142caaf672.png)

#### Validations
> + pack means `nuget pack`
> + push means `nuget push`
> + upload means https://www.nuget.org/packages/manage/upload 

* Size and other limitations for the md file that exists on NuGet.org still apply.
  * md file violates nuget.org size limits -> error on push, upload.
  * md contains unsupported syntax (Only core CommonMark features supported (no tables), and no inline HTML, injects “no follow” on all links) -> warn on pack. error on push, upload.
* documentation is null -> warn on pack, push, upload. (users should remove that tag or add a meaningful readme.md file)


***
[Back to parent spec](https://github.com/NuGet/Home/wiki/Packaging-Icon,-License-and-Documentation-within-the-nupkg)

[Packaging Icon within the nupkg](https://github.com/NuGet/Home/wiki/Packaging-Icon-within-the-nupkg)

[Packaging License within the nupkg](https://github.com/NuGet/Home/wiki/Packaging-License-within-the-nupkg)