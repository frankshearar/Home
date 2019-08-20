* Status: **Incubation**
* Author(s): [Anand Gaurav](https://github.com/anangaur) ([@adgrv](https://twitter.com/adgrv))

## Issue
Flag vulnerable packages [#8087](https://github.com/NuGet/Home/issues/8087)

## Problem
Today developers take dependency on a bunch of packages directly or indirectly (through transitive dependencies) and they have no way to understand if any of their dependencies bring in any known vulnerabilities. 

## Out of scope
The following are out of scope for this feature:
* Ability to report a vulnerability on nuget.org - This is a follow-up feature

## Solution
The vulnerability info needs to be shown for the following:
* `audit` or `list --vulnerable` command
* On Visual Studio NuGet Package Manager `Installed` tab 
  - Top-level dependencies
  - Transitive dependencies
* Upon `restore` - show minimal info/warning with a message to run another command for detailed output.
  - ⚠️`Warn` for `critical` vulnerabilities
  - ℹ️ `Info` for other levels
  - An option to opt-out of vulnerability check
* Upon `Install`
  - CLI can rely upon the subsequent `restore` operation for messaging
  - Visual Studio upon Install
* On Visual Studio NuGet Package Manager's package details for Browse | Installed | Updates tabs
* On NuGet.org 
  - On search list if the latest version of the package contains a vulnerability
  - On search list if the latest version of the package depends upon a vulnerable package (transitive - any level)
  - On package details page

In addition, we need the following:
* **[P1]** `why` command to understand why a package is shown as a dependency for my project. This is useful when I see a vulnerability in my package graph but I am not sure why and how this package (transitive) is showing up in the graph.
* **[P1]** An API that lists all vulnerable NuGet.org packages with the vulnerability information.
* **[Future]** A way to specify another source for vulnerability information


