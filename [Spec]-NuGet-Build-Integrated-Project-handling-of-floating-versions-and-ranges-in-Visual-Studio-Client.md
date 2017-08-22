## Issues

[#3788 - NETCore PackageReference with a star is displayed under the updates tab](https://github.com/NuGet/Home/issues/3788)

[#3101 - NuGet Visual Studio UI shows floating versions incorrectly](https://github.com/NuGet/Home/issues/3101)

[#3109 - NuGet UI cannot interact with version ranges with no minimum](https://github.com/NuGet/Home/issues/3109)

## Problem

In a build integrated NuGet project a user can specify a range or a floating version as a package version. The current behavior of the package manager UI inside of Visual Studio has no support for this. If a user indicates a floating version or a range in any of the packages (e.g. `10.0.*`) the UI will take its minimum version possible (e.g. `10.0.0`) and show it as if it is installed, even though the actual resolved version could be different. In the following example, the package reference specifies `3.7.*`, and the resolved version (found in the assets file) is `3.7.1`, nevertheless the UI shows that the installed version is `3.7.0`, which is misleading.

<img src="https://user-images.githubusercontent.com/2132567/29295602-215c2944-810a-11e7-9564-c64eaaf076e1.png" width="98%">

_UI for list item at project level_

<img src="https://user-images.githubusercontent.com/2132567/29295614-35fdf36e-810a-11e7-8440-bb556d6aadb1.png" width="49%">
<img src="https://user-images.githubusercontent.com/2132567/29295617-3d360d6a-810a-11e7-9af6-68f4d4bb04b7.png" width="49%">

_UI for detail control at project level_

<img src="https://user-images.githubusercontent.com/2132567/29578232-6cf36196-8723-11e7-8b01-d41b52c181e9.PNG" width="49%">

_UI for detail control at solution level_


This scenario can also be problematic if the calculated version doesn't exist, because there are several incorrect indications of the package version (both at solution and at project level), and also the list item will not load the correct metadata for the package. In the example below the specified version was `10.0.*` and it was resolved to `10.0.3`, but version `10.0.0` doesn't exist for this package.

<img src="https://user-images.githubusercontent.com/2132567/29295888-caba7080-810b-11e7-84c2-8ecb3c229c19.png" width="98%">

_UI for list item at project level_

<img src="https://user-images.githubusercontent.com/2132567/29295892-d320d520-810b-11e7-8745-84b2f9a0e280.png" width="49%">

_UI for detail control at project level_

## Solution

The current behavior of the Visual Studio client at a project level with packages using non-floating versions is to show the version installed on the right on the list item, on the "Installed" box in the detail view and bold in the "Version" dropdown in the detail view for each dependency.

<img src="https://user-images.githubusercontent.com/2132567/29296146-43723f16-810d-11e7-8b4f-6d6b6b18bfcd.png" width="98%">

_UI for list item at project level_

The updates are always suggested based on this installed/resolved version.

<img src="https://user-images.githubusercontent.com/2132567/29296148-46a717ec-810d-11e7-8264-1bc1641191c2.png" width="98%">

_UI for list item at project level_

<img src="https://user-images.githubusercontent.com/2132567/29296153-4baa1064-810d-11e7-9325-7896a9867650.png" width="49%">
<img src="https://user-images.githubusercontent.com/2132567/29296150-49401f08-810d-11e7-8330-57451068131e.png" width="49%">

_UI for detail control at project level_

In the case of the solution level, the version installed for each project is shown in the project list, in the "Installed" box and bold in the "Version" dropdown in the detail view for each dependency.

<img src="https://user-images.githubusercontent.com/2132567/29578242-72ed6222-8723-11e7-9a27-942dcb1ca3ce.PNG" width="49%">

_UI for detail control at solution level_

Also when a package is updated the version in the package reference is updated with the version just installed.

The proposed solution involves updating the client (in both, the project and the solution level UI) to get the actual resolved version for each package from the assets file instead of calculating the minimum version, therefore the version number shown will always portrait the actual version installed.

For the manager UI at the project level, this involves updating the version on the right in the list item, the version on the "Installed" section of the detail view and the bold version of the "Version" dropdown in the detail view to show the actual version resolved, not a minimum calculated by the client.

In the case of the manager UI at the solution level, this involves updating the version on the project list for each project, the "Installed" section of the detail view and the bold version of the "Version" dropdown in the detail view to show the version resolved.

This solution also involves that when a package is updated and the package reference specifies a floating version or a range, this reference will only be updated if the new version installed doesn't fall inside of the range previously specified.

Taking this updates in consideration, there are some scenarios that are relevant to mention the desired user experience involved. These scenarios are presented below.

### Scenarios

- There is no `project.assets.json` file.
- The project has multiple target frameworks and more than one of these have the same package with different versions.
- Updating a reference and running restore when the UI is open (changing resolved version)

Each of this scenarios propose different problems with this solution, the following sections explain them in a more detailed way.

### No `project.assets.json` file


### Multiple target frameworks with different versions of the same package


### Changing resolved version of a package while UI open

