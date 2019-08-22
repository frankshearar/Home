* Status: **Incubation**
* Author(s): [Anand Gaurav](https://github.com/anangaur) ([@adgrv](https://twitter.com/adgrv))

## Issue
Flag vulnerable packages [#8087](https://github.com/NuGet/Home/issues/8087)

## Problem
Today developers take dependency on a bunch of packages directly or indirectly (through transitive dependencies) and they have no way to understand if any of their dependencies bring in any known vulnerabilities. 

## Scope
There are multiple scenarios for flagging and reporting vulnerabilities. This feature will enable a subset of those as discussed below.

#### ✔️ In scope
* A CLI command to list vulnerabilities in a project/solution project graph
* Show vulnerability in the package graph on `Installed` tab of Visual Studio
  - Show vulnerabilities in transitive packages
* Enable consumers to know vulnerability at the time of Installation of a package
* Enable `restore` to Inform or Warn:
  - Warn (⚠️) for `critical` and `high` impact vulnerabilities (minimal info + command/way to uncover detailed information)
  - Info (ℹ️) for lower impact vulnerabilities (minimal info + command/way to uncover detailed information)
  - Option to enable/disable warnings for the various impact levels of the vulnerabilities
* Show vulnerability information in packages to their publishers on nuget.org
  - Notify package publishers for any new vulnerability detected in one-more of their packages

#### ❓ Open questions
* For `restore`, should it be opt-in or opt-out (preferred for security features)?
 
#### ❌ Out of scope
*Backlog: Follow-up feature to be implemented after the current one is implemented*

*Discuss: Feature that needs further discussion*

| Feature | State |
|:--- |:--- |
| A way to specify another source for vulnerability information | Backlog |
| `why` command to understand why a package is shown as a dependency for my project | Backlog |
| Ability to report a vulnerability on nuget.org | Discuss |
| An API that lists all vulnerable nuget.org packages with the vulnerability information | Discuss |
| Show vulnerability information as part of search results | Discuss |
| Show vulnerability information publicly for each package like deprecation info on the package details | Discuss |

## Solution
Vulnerability information can be shown in multiple scenarios:
### Using a CLI command 

### Upon `Install`

### On Visual Studio PM UI 

### Upon `restore`

### On nuget.org, for package publishers



