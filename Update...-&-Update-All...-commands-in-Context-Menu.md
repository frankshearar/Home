![image](https://user-images.githubusercontent.com/14800916/72940790-59a00e00-3d24-11ea-89bd-1bd04ab2aa76.png)

## Issue
The work for this feature and the discussion around the spec is tracked here:
Update... & Update All... commands in Context Menu [#8956](https://github.com/NuGet/Home/issues/8956)

## Problem
Installed packages are viewable in the solution explorer, but the context menu associated with right-clicking the packages and packages folder is very limited. There should be an option to update a package and update all packages through the context menu.

## Who is the customer?
NuGet package users.

## Evidence
From Developer community suggestion: https://developercommunity.visualstudio.com/content/idea/622379/add-manage-nuget-packages-to-context-menu.html

**Add Manage NuGet packages to context menu for NuGet node [5049](https://github.com/NuGet/Home/issues/5049)**

## Key Scenarios
The key scenarios we want to enable include:
* **Single Package Update:** Show "Update..." option in context menu for individual packages. This will open up the package manager UI to the update tab with the chosen package selected.
![image](https://user-images.githubusercontent.com/15097183/73973235-0ce93500-48d7-11ea-9ce9-cc662f532dcc.png)
* **Multi-Package Update:** Show "Update..." option in context menu when multiple packages are selected. This will open up the package manager UI to the update tab with all chosen packages selected.
![image](https://user-images.githubusercontent.com/15097183/73979134-791d6600-48e2-11ea-87c3-2ba093d0e3b4.png)
* **Update All Packages:** Show "Update..." option in context menu for top level packages folder. This will open up the package manager UI to the update tab with all packages selected.
![image](https://user-images.githubusercontent.com/15097183/73978401-1bd4e500-48e1-11ea-8ace-7dedc5edd71b.png)

## Design Details

### Finalized
Q: What is shown when update(s) aren't available for the selection?
A: The "Update..." item will still be available. Click on it will navigate to the "Updates" tab in the package manager UI but will have no packages selected since the selected package(s) won't exist on that page.

Q: How will a user know if an update is available?
A: In this iteration, there will not be an indicator of an update being available

### In Review
Q: What happens when a user selects an update for a pre-release package, but pre-release packages are being filtered out?
A: Set filter so that package is visible in update tab.

## Out of scope

* Indicator for number of updates available in solution explorer; number displayed next to the top level packages folder.
* Indicator that individual packages are eligible for updates located next to the respective package.
* Remove "Update..." option if the selected subset of packages have no updates available
* Drop-down menu for Updates allowing "Update to latest version," "Update to latest minor version," "Update..."
* Change "Remove" option to "Uninstall." (in review)
* Remove "..." from "Manage NuGet Packages..." (in review)

#### North Star
This example is what the UI on VS for Mac currently looks like in the solution explorer for the "Packages" folder. It includes the majority of planned UI changes that are out of scope for the first iteration of changes outlined in this spec.

<img width="529" alt="Screen Shot 2020-01-02 at 11 49 31 AM" src="https://user-images.githubusercontent.com/2878341/71843808-8cd27400-3079-11ea-93cd-9e065a18ee86.png">

