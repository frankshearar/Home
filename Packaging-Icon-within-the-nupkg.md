* Status: **Reviewed**
* Author(s): [Karan Nandwani](https://github.com/karann-msft) ([@karann9](https://twitter.com/karann9))

### Issue
The work for this feature and the discussion around the Icon specific spec is tracked by **Package Icon should be able to come from inside the package [#352](https://github.com/NuGet/Home/issues/352)**

### Icon

* The icon image will be a part of the nupkg
* New nuspec property `<icon>icon.png</icon>`
  * is a path to an image file inside the package
  * Supported formats - jpg, jpeg, png
  * the user will need to ensure the image file is packed by adding a files element. E.g. `<file src="image\icon.png" target="" />`
* If `icon.png` or `icon.jpg` is present at the folder/project root, and icon property is not present in the nuspec/project file, nuget pack should pack that file as the package icon.
* Browse from NuGet.org
  * During package ingestion, nuget.org will extract and store the icon.
  * For existing packages, the gallery would read the iconurl from the nuspec, fetch the image and store it.
  * For search, registration metadata, and v2 API (basically any service that customers hit where we expose the icon URL), the icon URL would be replaced with URL to the gallery hosted image location. These are the APIs that are called by Visual Studio when a user is on the "Browse" tab of the package manager UI. No client change would be required for this part. 
* Installed packages
  * Client will read the image from the global packages folder
  * For new packages, on package installation, the icon would also be extracted to the global packages folder and will be used as the image source to display the icon for installed packages.
  * For existing packages, on package installation, cache the icon to the global packages folder during installation (installing using dotnet CLI, will do the right thing to ensure the icon is available to the PMUI when the project is opened in VS.)
* Browse from folder based feed (including VS Offline Packages source) - read the icon from the nupkg. (similar to how we read the nuspec for such packages today)
* Browse from the fallback folder - the packages would be pre-extracted. Read the icon from the extracted location.
* Project properties
  * Replace `Icon URL` with `Icon` with a file picker.
  * On selecting a file on disc, the value updates with the relative path to the file.
  * In the project file `<PackageIcon>icon.png</PackageIcon> ` as well as
    ```
    <PropertyGroup>
      ...
      <PackageIcon>icon.png</PackageIcon>
      ...
    </PropertyGroup>
    <ItemGroup>
      ...
      <None Include="images\icon.png" Pack="true" PackagePath=""/>
      ...
    </ItemGroup>
    ```

  * Set the PackageIcon property in the project file to icon file name

#### Other considerations
* NuGet.org admin view to remove the package icon and block it from being displayed on NuGet.org or in VS during browse
* Size recommendation - 128x128 (on nuget.org we use a max of 76.5x76.5 and in VS 32x32)

#### Validations
* If the icon file is not found at the path listed under `icon` (this will also prevent template values from making into the system)
  * Error `nuget pack` and `nuget push` 
  * Fail package validation on NuGet.org
  * PMUI fail silently when browsing packages from a folder based feed
* If `iconurl` is present
  * Error `nuget pack` and `nuget push` 
  * Fail package validation on NuGet.org (synchronous validation)

#### Stretch goal

[Back to parent spec](https://github.com/NuGet/Home/wiki/Packaging-Icon,-License-and-Documentation-within-the-nupkg)

[Packaging License within the nupkg](https://github.com/NuGet/Home/wiki/Packaging-License-within-the-nupkg)

[Packaging Documentation within the nupkg](https://github.com/Home/Engineering/wiki/Packaging-Documentation-within-the-nupkg)