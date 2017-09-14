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

###Package compatibility
A package is deemed incompatible if it contains no assets that can be consumed if it were to be installed to the host project.

###Discover "Include incompatible" 
By default, incompatible packages will be excluded from search results.




