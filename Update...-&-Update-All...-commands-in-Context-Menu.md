![image](https://user-images.githubusercontent.com/14800916/72940790-59a00e00-3d24-11ea-89bd-1bd04ab2aa76.png)

## Issue
The work for this feature and the discussion around the spec is tracked here:
**Update... & Update All... commands in Context Menu [#8956](https://github.com/NuGet/Home/issues/8956)**

## Problem
Installed packages are viewable in the solution explorer, but the context menu associated with right-clicking the packages and packages folder is very limited. There should be an option to update a package and update all packages through the context menu.

## Who is the customer?
NuGet package users.

## Evidence
From Developer community suggestion: https://developercommunity.visualstudio.com/content/idea/622379/add-manage-nuget-packages-to-context-menu.html

**Add Manage NuGet packages to context menu for NuGet node [5049](https://github.com/NuGet/Home/issues/5049)**

## Key Scenarios
The key scenarios we want to enable include:
* Show "Update..." option in context menu for individual packages. This will open up the package manager UI to the update tab with the chosen package selected.
* Show "Update..." option in context menu when multiple packages are selected. This will open up the package manager UI to the update tab with all chosen packages selected.
* Show "Update All..." option in context menu for packages folder. 

### Out of scope

* Indicator for number of updates available in solution explorer; number displayed next to the top level packages folder.
* Indicator that individual packages are eligible for updates located next to the respective package.
* Drop-down menu for Updates allowing "Update to latest version," "Update to latest minor version," "Update..."
* Change "Remove" option to "Uninstall."

## Solution

* Add "Update All..." to "Packages" node context menu.
* Add "Update..." to package context menus when a single or multiple packages are selected.

## FAQs

* What is shown when update(s) aren't available for the selection?
Update option is grayed out.
* What happens when a user selects an update for pre-release packages, but pre-release packages are being filtered out?
Set filter so that package is visible in update tab.
* How will a user know if an update is available?

