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
By default, incompatible packages will be excluded from search results. On first search, a message would apprise the user that results are being filtered. The message can be dismissed with the option to never show it again.

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0010.png)

An info message would be displayed if a search returns no results since incompatible results have been filtered out.

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0020.png)

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0030.png)

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0040.png)

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0050.png)

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0060.png)

![](https://github.com/NuGet/Home/blob/dev/resources/PA/PA_0070.png)

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






