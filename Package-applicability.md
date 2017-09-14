Status: **Incubation**

## Issue
The work for this feature and the discussion around the spec is tracked here - **Context aware search and surface project applicability information [#5725](https://github.com/NuGet/Home/issues/5725)**

## Problem
As a package consumer:
* I have no way to filter to only those packages that will work with my project.
* When a package fails to install, I don't have a straight-forward way to know why it failed.
* What steps can I take to either install the selected version of a package or find a compatible version of the package.

## Who is the customer?
All NuGet package consumers that use the NuGet package manager UI to manage NuGet packages

## Solution
>Note: The workflow below focuses on the end-user experience and does not go into implementation details. Once, the experience has been finalized, we'll discuss about the implementation.

### Package compatibility
A package is deemed incompatible if it contains no assets that can be consumed if it were to be installed to the host project.

### Discover "Include incompatible" filter
By default, incompatible packages will be excluded from search results. On search, a message would inform the user that results are being filtered. The message can be dismissed with the option to not show it again.

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0010.png)

An info message would be displayed if a search returns no results since incompatible results have been filtered out. 

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0020.png)

### Search results when "Include incompatible" is checked
When "Include incompatible" is checked, all results are displayed.
Packages where no version of that package is compatible with the current project, are grayed out. In the example below, all search results returned are incompatible with `ClassLibrary1` which is a .NET Standard 2.0 class library.

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0030.png)

### Safeguarding the user from installing an incompatible package
An incompatible package is blocked from being installed. The "Learn more" link directs to documentation explaining why the installation is blocked, possible reasons, and what actions can be taken.

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0040.png)

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0050.png)

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0060.png)

### Latest compatible version selected by default instead of latest available version
Now consider the same search was performed in context of `App1` which is a UWP project with target platform Min version of Windows 10 November Update (10.0 Build 10586). By default, the **latest compatible version** of the package is selected, which in this case is v1.4.1 because that was the last version of the package where 10.0.10586 was supported.

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0070.png)

Expanding the version dropdown reveals all available versions and, at a glance, shows which version are supported.

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0080.png)



![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0090.png)

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0100.png)

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0110.png)

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0120.png)

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0130.png)

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0140.png)

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0150.png)

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0160.png)

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0170.png)

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0180.png)






