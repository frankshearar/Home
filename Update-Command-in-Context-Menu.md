## Issue
The work for this feature and the discussion around the spec is tracked here:
Update... & Update All... commands in Context Menu [#8956](https://github.com/NuGet/Home/issues/8956)

## Problem
Installed packages are viewable in the solution explorer, but the context menu associated with right-clicking the packages and packages folder is very limited. There should be an option to update packages through the context menu.

## Who is the customer?
Visual Studio NuGet package consumers.

## Evidence
From Developer community suggestion: https://developercommunity.visualstudio.com/content/idea/622379/add-manage-nuget-packages-to-context-menu.html

**Add Manage NuGet packages to context menu for NuGet node [5049](https://github.com/NuGet/Home/issues/5049)**

## Scope

**Supported Project Systems:** SDK-Style Package References

**Supported Nodes:** "Packages" top level folder and all child package elements

![image](https://user-images.githubusercontent.com/15097183/74058955-f6f17800-499b-11ea-9056-7712620a5b3f.png)

**Behavior Change:** Changes constrained to update ability in context menus for "Packages" folder and child elements

## Key Scenarios
The key scenarios we want to enable include:

**Single Package Update:** Show "Update..." option in context menu for individual packages. This will open up the package manager UI to the update tab with the chosen package selected.
![image](https://user-images.githubusercontent.com/15097183/73993637-a29cb880-4907-11ea-9314-dfe0f9a55343.png)
* Red box indicates where the focus will default to for accessibility.

**Multi-Package Update:** Show "Update..." option in context menu when multiple packages are selected. This will open up the package manager UI to the update tab with all chosen packages selected.
![image](https://user-images.githubusercontent.com/15097183/73994132-58b4d200-4909-11ea-9c37-c0eafb3396e9.png)
* Red box indicates where the focus will default to for accessibility.
* Not all selected packages appear in "Updates" tab because Moq did not have an available update.

**Update All Packages:** Show "Update..." option in context menu for top level packages folder. This will open up the package manager UI to the update tab with all packages selected.
![image](https://user-images.githubusercontent.com/15097183/73994028-f1971d80-4908-11ea-9442-43665c8c7352.png)
* Red box indicates where the focus will default to for accessibility.
* Not all packages appear in "Updates" tab because not all packages have available updates.

## Design Details FAQ

**Q: What is shown in the context menu when update(s) aren't available for the selection?**

A: The "Update..." item will still be available. Click on it will navigate to the "Updates" tab in the package manager UI but will have no packages selected since the selected package(s) won't exist on that page.

**Q: How will a user know if an update is available?**

A: In this iteration, there will not be an indicator of an update being available in the solution explorer.

**Q: What happens when a user selects "Update..." for a package, multiple packages, or all packages when no updates are available for the selection?**

A: The "Update" tab in the package manager UI will still be brought up, but no packages will be pre-selected/checked.

**Q: What happens when not all the packages in the selected subset of packages have an available update?**

A: Only the packages with existing updates will be pre-selected/checked in the "Update" tab.

**Q: What happens when a user selects an update for a pre-release package, but pre-release packages are being filtered out?**

A: Respect the user's existing "include prerelease" setting. If they want to see the prerelease update, they will enable "include prerelease" manually.

## Out of scope

* Indicator for number of updates available in solution explorer; number displayed next to the top level packages folder.
* Indicator that individual packages are eligible for updates located next to the respective package.
* Remove "Update..." option if the selected subset of packages have no updates available
* Remove "..." from "Manage NuGet Packages..."
* Drop-down menu for Updates allowing "Update to latest version," "Update to latest minor version," "Update..." (tentative)
* "Add Package" option in context menu (tentative)

#### North Star
This is roughly the ideal look of the "Packages" node we are striving for upon completion of the steps outlined in this spec as well as the changes listed in the "Out of scope" section that will be executed in future iterations. The north star of the context menu will require further efforts with UX designers and customer development.

<img width="529" alt="Screen Shot 2020-01-02 at 11 49 31 AM" src="https://user-images.githubusercontent.com/2878341/71843808-8cd27400-3079-11ea-93cd-9e065a18ee86.png">

