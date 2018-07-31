* Status: **Reviewing**
* Author(s): [Karan Nandwani](https://github.com/karann-msft) ([@karann9](https://twitter.com/karann9))

### Issue
The work for this feature and the discussion around the Documentation specific spec is tracked by **Nuspec - documentation / readme url [#6873](https://github.com/NuGet/Home/issues/6873)**

### Documentation

* New nuspec property `<documentation src="documentation.md" />`
  * target defaults to package root
  * Supported formats - md
  * On pack, strip source value, append the target value with the source file name - `<documentation target="documentation.md"/>`. (this is to help the gallery and the client know the path to the icon file in the package and the file name.extension. Allows for extension validation, etc.)
  * nuget spec will add 
    > `<!-- e.g. <documentation src="documentation.md"/>. Learn more at https://aka.ms/nugetPackageDocumentation-->` <br>
    > `<documentation src=""/>`
  * package submitted to NuGet.org - target should not be null
* Browse from NuGet.org
  * During package ingestion, nuget.org will extract and validate the md, and update the package details page with the content.
  * The documentation url would be served by the gallery. For nuget.org this will link to the package details page that contains documentation (and that section will be expanded).
* Installed packages/folder based feeds/fallback folder
  * Client will provide a link to open the documentation file from the nupkg/global packages folder/extracted location
  * Clicking on the link will open the file in the default application associated with that extension
  * Client will do the same validations and security checks as nuget.org before displaying md files.

#### Other considerations
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