## Issue

The investigation started with [#3788 - NETCore PackageReference with a star is displayed under the updates tab](https://github.com/NuGet/Home/issues/3788).

## Problem

The package manager client inside Visual Studio has no support for floating versions in the UI. If a user indicates a floating version in any of the packages (e.g. `10.0.*`) the UI will take the minimum version possible for the floating range (e.g. `10.0.0`) and show it as if it is installed, even though the actual resolved version could be different. In the following example, the package reference specifies `3.7.*`, and the resolved version (found in the assets file) is `3.7.1`, nevertheless the UI shows that the installed version is `3.7.0` which is misleading.

<img src="https://user-images.githubusercontent.com/2132567/29295602-215c2944-810a-11e7-9564-c64eaaf076e1.png" width="98%">

_UI for list item_

<img src="https://user-images.githubusercontent.com/2132567/29295614-35fdf36e-810a-11e7-8440-bb556d6aadb1.png" width="49%">
<img src="https://user-images.githubusercontent.com/2132567/29295617-3d360d6a-810a-11e7-9af6-68f4d4bb04b7.png" width="49%">

_UI for detail control_

This scenario can also be problematic if the calculated version doesn't exist, because neither the list item nor the detail control will display the correct metadata for the package. In the example below the specified version was `10.0.*` and it was resolved to `10.0.3`, but version `10.0.0` doesn't exist for this package.

<img src="https://user-images.githubusercontent.com/2132567/29295888-caba7080-810b-11e7-84c2-8ecb3c229c19.png" width="98%">

_UI for list item_

<img src="https://user-images.githubusercontent.com/2132567/29295892-d320d520-810b-11e7-8745-84b2f9a0e280.png" width="49%">

_UI for detail control_

The following scenarios where identified as relevant to take into account in the proposed solution.

### Scenarios

- The latest version available falls inside the floating range specified and the resolved version of the package is the same as the latest available.
- The latest version available falls inside the floating range specified and the resolved version of the package is older than the latest available.
- The latest version available doesn't fall inside the floating range specified but the resolved version of the package is the max of the range specified.
- The latest version available doesn't fall inside floating range specified but the resolved version of the package is not the max of range specified.

## Solution

The current behavior of the Visual Studio client with packages using non-floating versions is to show the version installed as on the right on the list item, on the "Installed" box in the detail view and bold in the "Version" dropdown in the detail view for each dependency.

<img src="https://user-images.githubusercontent.com/2132567/29296146-43723f16-810d-11e7-8b4f-6d6b6b18bfcd.png" width="98%">

_UI for list item_

The updates are always suggested based on this installed/resolved version.

<img src="https://user-images.githubusercontent.com/2132567/29296148-46a717ec-810d-11e7-8264-1bc1641191c2.png" width="98%">

_UI for list item_

<img src="https://user-images.githubusercontent.com/2132567/29296153-4baa1064-810d-11e7-9325-7896a9867650.png" width="49%">
<img src="https://user-images.githubusercontent.com/2132567/29296150-49401f08-810d-11e7-8330-57451068131e.png" width="49%">

_UI for detail control_

The proposed solution involves updating the client to get the actual resolved version for each package from the assets file instead of calculating the minimum version, therefore the version number shown will always portrait the actual version installed.

For the four scenarios mentioned above the version on the right in the list item, the version on the "Installed" section of the detail view and the bold version of the "Version" dropdown in the detail view should show the actual version resolved, not a minimum calculated by the client. Also, each scenario needs a different experience when an update is available.

### The latest version available falls inside the floating range specified and the resolved version of the package is the same as the latest available

The user has the latest version, the client should not show an update available for this package (the same way as if the user had directly installed the latest version).

### The latest version available falls inside the floating range specified and the resolved version of the package is older than the latest available

Show the update icon and have the blue number over the "Updates" tab. If the user decides to update to the new version the package reference should not be modified because it falls inside of the current range.

### The latest version available doesn't fall inside the floating range specified but the resolved version of the package is the max of the range specified

Show the update icon and have the blue number over the "Updates" tab. If the user decides to update to the new version the package reference should be modified and the user should be warned.

### The latest version available doesn't fall inside floating range specified but the resolved version of the package is not the max of range specified

Show the update icon and have the blue number over the "Updates" tab. Depending on what version the user selects the behavior should follow one of the scenarios above.

## Open Questions
- When using floating versions, the restore command should update the package if a new version inside of the proposed range is available. Should the manager UI treat a new version inside the floating range as a regular update?

  * [nkolev92] I do not think that these updates should be treated as regular updates. In my opinion this would be misleading, I think we can have some sort of an indicator that the package is not "up to date" in the installed tab or something, but we should not display them in the Updates tab. 
Technically a bit more complex, but we can consider displaying a nice "NU warning" that the package they have restored is not up to date, and tell them they should rerun restore to get the latest package. The tricky part here is figuring out the trigger for that warning, such as opening the UI etc. 

- Should there be a way to indicate a package is using a floating version and also indicate which version the package resolved to?
- Does the client UI needs a way to let the user specify an installation with floating version from the UI?
