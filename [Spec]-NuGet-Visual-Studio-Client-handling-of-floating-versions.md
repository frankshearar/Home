## Issue

The investigation started with [#3788 - NETCore PackageReference with a star is displayed under the updates tab](https://github.com/NuGet/Home/issues/3788), but is related to [#5714 - [Idea] Enhance "available updates" experience to show more critical updates](https://github.com/NuGet/Home/issues/5714) for enhancing the update package experience.

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

The proposed solution involves updating the client to get the actual resolved version for each package from the assets file instead of calculating the minimum version, therefore the version number shown will always portrait the actual version installed. This solution leaves some questions open about how to handle updates in packages with a floating version.

As [@joelverhagen](https://github.com/joelverhagen) mentioned in [#5714](https://github.com/NuGet/Home/issues/5714) there are different scenarios for updating a package and each of them has a different urgency. NuGet Client should provide different signals to show the different types of updates. To clarify what each signal means a tooltip message can be provided. 

For the four scenarios mentioned above the version on the right in the list item, the version on the "Installed" section of the detail view and the bold version of the "Version" dropdown in the detail view should show the actual version resolved, not a minimum calculated by the client. Also, each scenario needs a different experience when an update is available.

### The latest version available falls inside the floating range specified and the resolved version of the package is the same as the latest available

The user has the latest version, the client should not show an update available for this package (the same way as if the user had directly installed the latest version).

### The latest version available falls inside the floating range specified and the resolved version of the package is older than the latest available

Show another icon for update (e.g. green up arrow) and specify with a tooltip that an update is available by doing a restore, but don't show it NuGet's updates tab.

<img src="https://user-images.githubusercontent.com/2132567/29296155-4de2db0e-810d-11e7-8c70-f46fc0c1341d.png" width="98%">

### The latest version available doesn't fall inside the floating range specified but the resolved version of the package is the max of the range specified

Show the update icon and have the blue number over the "Updates" tab. The user can then decide if they want to update to a new version and have NuGet update the package reference to reflect this action.

### The latest version available doesn't fall inside floating range specified but the resolved version of the package is not the max of range specified

Show only one of the signals of updates mentioned in the above scenarios (either update by restoring or update by UI). **Which one is more important?**

## Open Questions
- When using floating versions, the restore command should update the package if a new version inside of the proposed range is available. Should the manager UI treat a new version inside the floating range as a regular update?
- Should we show an update when there is a new version inside the floating range and the user hasn't ran restore?
- Should we shown an update for a latest version outside of the floating range?
- Should we a package is using a floating version and also indicate which version the package resolved to?
- How are we going to manage updates? Right now we run the update and then rewrite the references to show the current installed version, in the case of floating ranges we don't want to overwrite the references because they might fall in that same range.
- Do we want to have a way to let the user specify an installation with floating version from the UI?
