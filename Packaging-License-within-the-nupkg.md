* Status: **Reviewing**
* Author(s): [Karan Nandwani](https://github.com/karann-msft) ([@karann9](https://twitter.com/karann9))

### Issue
The work for this feature and the discussion around the License specific spec is tracked by **Package trust - Licenses [#4628](https://github.com/NuGet/Home/issues/4628)**

### License

* New nuspec property `<license type="expression | file" />`
  * Is an [SPDX license identifier](https://spdx.org/licenses/) or expression. E.g. `<license type="expression">Apache-2.0</license>`
  * Or is a path to a license file. E.g. `<license type="file">LICENSE.txt</license>`
  * Supported formats - md, txt
  * the user will need to ensure the license file is packed by adding a files elemenet. E.g. `<file src="licenses\LICENSE.txt" target="" />`
  * nuget spec will add license instead of licentUrl 
    > `<!-- e.g. <license type="MIT"/> or <license src="license.txt"/>. Note - you cannot specify both type and source. Learn more at https://aka.ms/nugetPackageLicense-->` <br>
    > `<license type=""/>`

#### SPDX identifier or expression
* Browse from NuGet.org
  * During the package ingestion, nuget.org will extract and store the SPDX expression at the package version level.
  * VS 2017 and above / NuGet.org - this information will be surfaced as "[LGPL-2.1-only](https://spdx.org/licenses/LGPL-2.1-only.htm) or [MIT](https://spdx.org/licenses/MIT.html)"
  * VS 2015 and older - this information will continue to be a URL. If it is a single identifier, the URL points directly to the SPDX license text. If it is a license expression, the URL points to a NuGet.org page that surfaces the information as "[LGPL-2.1-only](https://spdx.org/licenses/LGPL-2.1-only.htm) or [MIT](https://spdx.org/licenses/MIT.html)"
  * Clicking on the license in VS opens the link in the browser.
  * (engineering call if we need a change to the protocol or if the client can parse the URL to surface this information accurately)
  * For existing packages, continue to serve license url from the nuspec, as we do today (nuget.org will stop accepting new packages with licenseurl, explained further in the roll-out plan)
* Installed packages/folder based feeds/fallback folder
  * VS 2017 and above
    * Client will read the SPDX expression from the nuspec and this information will be surfaced as "[LGPL-2.1-only](https://spdx.org/licenses/LGPL-2.1-only.htm) or [MIT](https://spdx.org/licenses/MIT.html)".
    * For existing packages, fall back to displaying license URL the way we do today
  * VS 2015 and older
    * At pack time, we will insert a license URL in the nuspec which will point to https://aka.ms/deprecateLicenseUrl (validations in place to ensure no other url can be used). Pack will provide an info message that this is being done to maintain compat. The docs will explain to the user why they ended up on that page and what can they do figure out the package license which is essentially navigating to the nuget.org package details page.
    * Stretch goal (out of scope for v1) - the client knows how to generate a URL that NuGet.org serves for the browse scenario and populate PackageLicenseUrl with that url. 
    * For existing package, no change in behavior

#### License file inside the pacakge

* Browse from NuGet.org
  * During the package ingestion, nuget.org will extract the license and host it on nuget.org. For search, registration metadata, and v2 API (basically any service that customers hit where we expose the license URL), the license URL would be replaced with URL to the gallery hosted license.
  * Clicking on the license link on the package details page navigates to the nuget.org hosted license page for that package version https://www.nuget.org/packages/Newtonsoft.Json/11.0.2/license
  * Stretch goal (out of scope for v1) - Attempt to parse the license file to determine license type and expose that information on nuget.org and in the API. If the file contains a license that can be replaced with an SPDX identifier, surface that information to the author.
* Installed packages/folder based feeds/fallback folder
  * VS 2017 and above
    * Client will provide a link to open the license file from the nupkg/global packages folder/extracted location
    * Clicking on the link will open the file in the default application associated with that extension
    * For existing packages, fall back to displaying license URL the way we do today
  * VS 2015 and older - same as what we do in case of SPDX identifier or expression

#### Project properties
* `License URL` will be removed from project properties and `PackageLicenseUrl` will be removed from the project file.
* Project properties will have `License type` (free text field that can take an SPDX expression)  and `License file` (free text field that can take a path to the license file on disk relative to the project file)
* `<PackageLicenseExpression>Apache-2.0</PackageLicenseExpression>`
* License file
```
    <PropertyGroup>
        <PackageLicenseFile>LICENSE.txt</PackageLicenseFile>
    </PropertyGroup>

    <ItemGroup>
        <None Include="licenses\LICENSE.txt" Pack="true" PackagePath="$(PackageLicenseFile)"/>
    </ItemGroup>
```

#### Validations
> + pack means `nuget pack`
> + push means `nuget push`
> + upload means https://www.nuget.org/packages/manage/upload 

* license is null or contains default value -> warn on pack, push, upload.
* licenseUrl is present and licenseUrl <> https://aka.ms/deprecateLicenseUrl (or client generated url to nuget.org as part of the stretch goal) -> error on pack, push, upload.
* Copyright field is null and license uses an SPDX identifier -> warn on pack, push, upload.
* when licenseUrl is added for the user -> info on pack

#### SPDX Expressions
  * Disjunctive OR Operator

    | nuspec  | VS / nuget.org |
    | ------------- | ------------- |
    | `<license type="expression">LGPL-2.1 OR MIT</license>`  | [LGPL-2.1-only](https://spdx.org/licenses/LGPL-2.1-only.htm) or [MIT](https://spdx.org/licenses/MIT.html) |

  * Conjunctive AND Operator

    | nuspec  | VS / nuget.org |
    | ------------- | ------------- |
    | `<license type="expression">LGPL-2.1 AND MIT</license>` | [LGPL-2.1-only](https://spdx.org/licenses/LGPL-2.1-only.htm) and [MIT](https://spdx.org/licenses/MIT.html) |

  * Exception WITH Operator

    | nuspec  | VS / nuget.org |
    | ------------- | ------------- |
    | `<license type="expression">GPL-2.0+ WITH Bison-exception-2.2</license>`  | [GPL-2.0+ w/ Bison-exception-2.2](https://spdx.org/licenses/GPL-2.0-with-bison-exception.html)  |

  * Combining multiple operators with parenthesis

    | nuspec  | VS / nuget.org |
    | ------------- | ------------- |
    | `<license type="expression">LGPL-2.1-only OR (GPL-2.0+ WITH Bison-exception-2.2 AND MIT</license>` | [LGPL-2.1-only](https://spdx.org/licenses/LGPL-2.1-only.htm) or ([GPL-2.0+ w/ Bison-exception-2.2](https://spdx.org/licenses/GPL-2.0-with-bison-exception.html) and [MIT](https://spdx.org/licenses/MIT.html)) | 


***
[Technical spec](https://github.com/NuGet/Home/wiki/Packaging-License-within-the-nupkg-(Technical-Spec))

[Back to parent spec](https://github.com/NuGet/Home/wiki/Packaging-Icon,-License-and-Documentation-within-the-nupkg)

[Packaging Icon within the nupkg](https://github.com/NuGet/Home/wiki/Packaging-Icon-within-the-nupkg)

[Packaging Documentation within the nupkg](https://github.com/NuGet/Home/wiki/Packaging-Documentation-within-the-nupkg)
