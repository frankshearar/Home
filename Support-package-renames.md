* Status: **Reviewing**
* Author(s): [Loïc Sharma](https://github.com/loic-sharma)
* Issue: [NuGetGallery#7896](https://github.com/NuGet/NuGetGallery/issues/7896)
* Type: Feature

## Problem Background

NuGet.org's search rankings is heavily influenced by packages' popularity. This leads to poor search quality when a popular package is renamed.

For example, here are popular packages that have been renamed:

Package | Replacements
-- | --
[FAKE](https://www.nuget.org/packages/FAKE/) | [fake-cli](https://www.nuget.org/packages/fake-cli/)
[iTextSharp](https://www.nuget.org/packages/iTextSharp/) | [itext7](https://www.nuget.org/packages/itext7/)
[Microsoft.Tpl.Dataflow](https://www.nuget.org/packages/Microsoft.Tpl.Dataflow/) | [System.Threading.Tasks.Dataflow](https://www.nuget.org/packages/System.Threading.Tasks.Dataflow/)
[Microsoft.SourceLink.Vsts.Git](https://www.nuget.org/packages/Microsoft.SourceLink.Vsts.Git) | [Microsoft.SourceLink.AzureRepos.Git](https://www.nuget.org/packages/Microsoft.SourceLink.AzureRepos.Git/)
[EntityFramework.Extended](https://www.nuget.org/packages/EntityFramework.Extended/) |[ Z.EntityFramework.Plus.EFCore](https://www.nuget.org/packages/Z.EntityFramework.Plus.EFCore/), [Z.EntityFramework.Plus.EF6](https://www.nuget.org/packages/Z.EntityFramework.Plus.EF6/)
[ManagedEsent](https://www.nuget.org/packages/ManagedEsent/) | [Microsoft.Database.ManagedEsent](https://www.nuget.org/packages/Microsoft.Database.ManagedEsent/)

## Target audience

Package authors that would like to rename their popular packages.

## Solution

### Add a "Rename" section to the "Manage Package" page

We will add a new section to the "Manage Package" page that lets the package's owners link the current package to its replacements:

![image](https://user-images.githubusercontent.com/737941/77343450-ea787300-6cee-11ea-95f1-935ffc452fd1.png)

Clicking on the `Learn more` link will lead the customer to a documentation page detailing explaining how to "rename" a package. The documentation will also explain the transfer popularity feature.

You can select any package as the `New package` that is different from the current package as long as it has at least one listed and non-deleted version. Furthermore, the `New package` may be a package owned by a different account.

The `Provide custom message` field is a free-form field to give consumers more context on the package rename. This field is not required.

Selecting `Transfer popularity` will split the popularity of the current package and transfer it equally to the new packages (more on that in the next section). As a result, the new packages will now be favored in search rankings over the replaced package. Given transferring the popularity "hurts" the current package's search rankings, a warning message is displayed if a `Transfer popularity` checkbox is selected.

We will only allow up to 5 new packages. Once you reach the limit, the `+ Add more` link will disappear:

![image](https://user-images.githubusercontent.com/737941/79151860-146a0600-7d80-11ea-9449-bae28e6f527e.png)

Saving will notify the user that it may take several hours for this change to propagate through our system:

![image](https://user-images.githubusercontent.com/737941/79152031-5c892880-7d80-11ea-9bf3-82f09bcd4835.png)

Opening the "Rename" section will show a message that popularity transfers are pending:

![image](https://user-images.githubusercontent.com/737941/79151940-3794b580-7d80-11ea-82d8-09d4347dd235.png)

### Show package renames on the "Display Package" page

Once you've marked your package as renamed, the "Display Package" page will notify consumers:

![image](https://user-images.githubusercontent.com/737941/79152134-83dff580-7d80-11ea-9948-8b94802fe84f.png)

If you chose a single `New package`, the message on the "Display Package" page will read:

	This package has been renamed

If you choose multiple `New package`s, the message on the "Display Package" page will read:

	This package has been renamed or split into new packages

Note that this message only appears on the renamed package. The new packages' "Display Package" page won't say anything!

### Popularity Transfer
Today, packages receive a popularity score based off their total downloads. These popularity scores influence search rankings: a package with a higher popularity score is more likely to be a top result.

Renamed packages will transfer a percentage of their downloads equally between the replacements that have `Transfer Popularity` checked. In other words, the replacement packages will have increased popularity scores. On the other hand, the renamed package will have a decreased popularity score. This transfer only affects popularity scoring, NuGet.org and Visual Studio will display packages' original downloads.

> **NOTE**: A package with both outgoing and incoming transfers will "ignore" the incoming downloads. The transferred incoming downloads are effectively lost.

### Visual Studio

Visual Studio's search rankings will reflect popularity transfers due to package renames. However, package renames information will NOT appear in Visual studio. This will be added in the future.