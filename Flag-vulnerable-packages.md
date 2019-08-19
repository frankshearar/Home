* Status: **Incubation**
* Author(s): [Anand Gaurav](https://github.com/anangaur) ([@adgrv](https://twitter.com/adgrv))

## Issue
Flag vulnerable packages [#8087](https://github.com/NuGet/Home/issues/8087)

## Problem
Today developers take dependency on a bunch of packages directly or indirectly (through transitive dependencies) and they have no way to understand if any of their dependencies bring in any known vulnerabilities. 

## Out of scope


## Solution
1. Show vulnerability info for packages with vulnerability:
  * NuGet.org package details page
    - Show detailed info for packages with vulnerabilities
    - Mention if one of the dependencies have vulnerability
  * Visual Studio package details for Browse | Installed | Updates tabs
2. Show vulnerable packages for
  * An `audit` or `list --vulnerable`
  * Visual Studio UI Installed tab 
    - Top-level dependencies
    - Transitive dependencies
  * `restore` - show minimal info/warning with a message to run another command for detailed output.
    - ⚠️`Warn` for `critical` vulnerabilities
    - ℹ️ `Info` for other levels
    - An option to opt-out of vulnerability check
  * Install
    - CLI can rely upon the subsequent `restore` operation for messaging
    - [Not MVP] Visual Studio upon Install
