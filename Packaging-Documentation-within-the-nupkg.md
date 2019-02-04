* Status: **Reviewing**
* Author(s): [Karan Nandwani](https://github.com/karann-msft) ([@karann9](https://twitter.com/karann9))

### Issue
The work for this feature and the discussion around the Documentation specific spec is tracked by **Nuspec - documentation / readme url [#6873](https://github.com/NuGet/Home/issues/6873)**

### Documentation

* The documentation file will be a part of the nupkg
* New nuspec property `<documentation>documentation.md</documentation>`
  * Is a path to a documentation file. E.g. `<documentation>docs.md</documentation>`
  * Supported formats - md
  * the user will need to ensure the documentation file is packed by adding a files elemenet. E.g. <file src="myAwesomeRepo\readme.md" target="readme.md" />
* If `documentation.md` is present at the folder/project root, and documentation property is not present in the nuspec/project file, nuget pack should pack that file as the package documentation.
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