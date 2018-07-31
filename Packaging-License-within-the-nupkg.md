### License

* New nuspec property `<license type="MIT" />`
  * Is an [SPDX license identifier](https://spdx.org/licenses/) or expression. E.g. `<license type="LGPL-2.1 OR MIT" />`
  * Or is a path to a license file. E.g. `<license src="license.txt" />`
  * Supported formats - md, txt
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
    * Stretch goal - the client knows how to generate a URL that NuGet.org serves for the browse scenario and populate PackageLicenseUrl with that url. 
    * For existing package, no change in behavior

#### License file inside the pacakge

* Browse from NuGet.org
  * During the package ingestion, nuget.org will extract the license and host it on nuget.org. For search, registration metadata, and v2 API (basically any service that customers hit where we expose the license URL), the license URL would be replaced with URL to the gallery hosted license.
  * Clicking on the license link on the package details page navigates to the nuget.org hosted license page for that package version https://www.nuget.org/packages/Newtonsoft.Json/11.0.2/license
  * Stretch goal - Attempt to parse the license file to determine license type and expose that information on nuget.org and in the API. If the file contains a license that can be replaced with an SPDX identifier, surface that information to the author.
* Installed packages/folder based feeds/fallback folder
  * VS 2017 and above
    * Client will provide a link to open the license file from the nupkg/global packages folder/extracted location
    * Clicking on the link will open the file in VS (if we hear feedback that this should not open in VS, we will pop a message the first time this happens, allowing the user to choose to open it with the default application associated with that extension, with the option to not show that message again)
    * Client will do the same security checks as nuget.org before displaying txt/md files.
    * For existing packages, fall back to displaying license URL the way we do today
  * VS 2015 and older - same as what we do in case of SPDX identifier or expression
* On pack, strip source value, append the target value with the source file name - `<license target="license.txt"/>`. (this is to help the gallery and the client know the path to the license file in the package and the file name.extension.)

#### Project properties
* `License URL` will be removed from project properties and `PackageLicenseUrl` will be removed from the project file.
* Project properties will have `License type` (free text field that can take an SPDX expression)  and `License file` (free text field that can take a path to the license file on disk relative to the project file)
* `<PackageLicense type="comes from the License type property", src="comes from the License file property", target="defaults to package root" />`
* On pack, strip source value, append the target value with the source file name. The license file will be placed at the package root and the nuspec property `license` will be set to `<license target="file name from the license file prop"/>` indicating that the icon license at the package root.
* PackageLicenseUrl - same as what we do for licenseUrl above


#### Validations
> + pack means `nuget pack`
> + push means `nuget push`
> + upload means https://www.nuget.org/packages/manage/upload 

* license is null or contains default value -> warn on pack, push, upload.
* licenseUrl is present and licenseUrl <> https://aka.ms/deprecateLicenseUrl (or client generated url to nuget.org as part of the stretch goal) -> error on pack, push, upload.
* Copyright field is null and license uses an SPDX identifier -> warn on pack, push, upload.
* when licenseUrl is added for the user -> info on pack

#### Client Impact matrix
The matrix below talks about scenarios where the license will be visible for a given scenario and client.<br>
Partial - aka.ms/deprecateLicenseUrl will be displayed for new packages. Existing packages won't be affected.<br>
Y - the intended license will be displayed for existing and new packages.

| Scenario | VS2015 | VS2017 | VS2019 |
| ------------- | ------------- | ------------- | ------------- |
| Browse packages from NuGet.org  | Y  | Y | Y |
| Browse packages from a folder based feed  | Partial (for new packages)  | Y  | Y  |
| Installed packages  | Partial (for new packages)  | Y  | Y  |


#### Roll-out plan
* `licenseurl` in the nuspec will be deprecated
  * Day 0 - Announce `licenseurl` is being deprecated (blog, tweet, etc.)
  * Day 0 - NuGet.org starts accepting packages with the `license` property
  * Day 0 - NuGet.exe ships that recognizes the `license` property
  * Day 0 - Start warning authors that `licenseurl` is being deprecated (licenseUrl is present and licenseUrl <> https://aka.ms/deprecateLicenseUrl -> warn on pack, push, upload.)
  * Day 20 - Announce `licenseurl` will not be accepted starting Day 30 deprecated
  * Day 30 - NuGet.org stops accepting packages with `licenseurl` (except if licenseurl = https://aka.ms/deprecateLicenseUrl)
  * Day 30 - NuGet.exe ships which errors on pack and push if licenseUrl is present and licenseUrl <> https://aka.ms/deprecateLicenseUrl
  * Day 30 - VS ships with the new NuGet client that is capable of surfacing the new license info.

#### SPDX Expressions
  * Disjunctive OR Operator

    | nuspec  | VS / nuget.org |
    | ------------- | ------------- |
    | `<license type="LGPL-2.1 OR MIT" />`  | [LGPL-2.1-only](https://spdx.org/licenses/LGPL-2.1-only.htm) or [MIT](https://spdx.org/licenses/MIT.html) |

  * Conjunctive AND Operator

    | nuspec  | VS / nuget.org |
    | ------------- | ------------- |
    | `<license type="LGPL-2.1 AND MIT" />` | [LGPL-2.1-only](https://spdx.org/licenses/LGPL-2.1-only.htm) and [MIT](https://spdx.org/licenses/MIT.html) |

  * Exception WITH Operator

    | nuspec  | VS / nuget.org |
    | ------------- | ------------- |
    | `<license type="GPL-2.0+ WITH Bison-exception-2.2" />`  | [GPL-2.0+ w/ Bison-exception-2.2](https://spdx.org/licenses/GPL-2.0-with-bison-exception.html)  |

  * Combining multiple operators with parenthesis

    | nuspec  | VS / nuget.org |
    | ------------- | ------------- |
    | `<license type="LGPL-2.1-only OR (GPL-2.0+ WITH Bison-exception-2.2 AND MIT" />` | [LGPL-2.1-only](https://spdx.org/licenses/LGPL-2.1-only.htm) or ([GPL-2.0+ w/ Bison-exception-2.2](https://spdx.org/licenses/GPL-2.0-with-bison-exception.html) and [MIT](https://spdx.org/licenses/MIT.html)) | 

#### Other considerations
* Document how to get license info from NuGet.org feed


***

[Back to parent spec](https://github.com/NuGet/Engineering/wiki/Packaging-Icon,-License-and-Documentation-within-the-nupkg)

[Packaging Icon within the nupkg](https://github.com/NuGet/Engineering/wiki/Packaging-Icon-within-the-nupkg)

[Packaging Documentation within the nupkg](https://github.com/NuGet/Engineering/wiki/Packaging-Documentation-within-the-nupkg)
