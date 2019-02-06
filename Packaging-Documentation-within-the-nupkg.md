* Status: **Reviewing**
* Author(s): [Karan Nandwani](https://github.com/karann-msft) ([@karann9](https://twitter.com/karann9))

### Issue
The work for this feature and the discussion around the Documentation specific spec is tracked by **Nuspec - documentation / readme url [#6873](https://github.com/NuGet/Home/issues/6873)**

### Documentation

* The documentation file will be a part of the nupkg
* New nuspec property `<documentation>documentation.md</documentation>`
  * Is a path relative to the package root to a documentation file inside the package`
  * Supported formats - md
  * the user will need to ensure the documentation file is packed by adding a files element. E.g. `<file src="myAwesomeRepo\readme.md" target="readme.md" />`
* If `documentation.md` is present at the folder/project root, and documentation property is not present in the nuspec/project file, nuget pack should pack that file as the package documentation.
* Browse from NuGet.org
  * During package ingestion, nuget.org will extract and validate the md, and update the package details page with the content.
  * Client will surface the documentation url served by the gallery
  * Clicking on the link will open the default browser and go to the package details page that contains documentation (and that section will be expanded).
* Installed packages/folder based feeds/fallback folder
  * Client will provide a link to open the documentation file from the nupkg/global packages folder/extracted location
  * Clicking on the link will open the file in the default application associated with `.md`extension
  * Client will do the same validations and security checks as nuget.org before displaying md files.
  ![image](https://user-images.githubusercontent.com/16904420/52244182-3f5ded80-2891-11e9-875c-beddcaf49e2b.png)

* Upload from NuGet.org package preview - documentation preview is rendered inline, similar to license file. (all the extra whitespace in the mock below is unintentional)
  ![image](https://user-images.githubusercontent.com/16904420/52312144-57e80980-295e-11e9-95cf-cc33ac1261b3.png)

* NuGet.org package edit page will no longer allow you to edit the documentation. Documentation will be immutable and the user must push a new version if they want to make changes to the documentation.

#### Project properties

![image](https://user-images.githubusercontent.com/16904420/52376505-254e1780-2a17-11e9-9bc8-a85258490c59.png)


```
    <PropertyGroup>
        <PackageDocumentationFile>documentation.md</PackageDocumentationFile>
    </PropertyGroup>

    <ItemGroup>
        <None Include="assets\documentation.md" Pack="true" PackagePath=""/>
    </ItemGroup>
```


#### Other considerations
* Similar to icon, NuGet.org admin view to remove the package documentation and block it from being displayed on NuGet.org or in VS during browse from NuGet.org.
![image](https://user-images.githubusercontent.com/16904420/52311447-d0010000-295b-11e9-89cc-b5142caaf672.png)

* Size and other limitations for the md file that exists on NuGet.org still apply.

#### Validations
> + pack means `nuget pack`
> + push means `nuget push`
> + upload means https://www.nuget.org/packages/manage/upload 

* md file violates nuget.org size limits -> error on push, upload.
* md contains unsupported syntax (Only core CommonMark features supported (no tables), and no inline HTML, injects “no follow” on all links) -> warn on pack, push, upload (security reasons)
* documentation is null -> warn on pack, push, upload. (users should remove that tag or add a meaningful doc.md file)


***
[Back to parent spec](https://github.com/NuGet/Home/wiki/Packaging-Icon,-License-and-Documentation-within-the-nupkg)

[Packaging Icon within the nupkg](https://github.com/NuGet/Home/wiki/Packaging-Icon-within-the-nupkg)

[Packaging License within the nupkg](https://github.com/NuGet/Home/wiki/Packaging-License-within-the-nupkg)