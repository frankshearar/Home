* Status: Planned
* Author(s): [Anand Gaurav](https://github.com/anangaur) ([@adgrv](https://twitter.com/adgrv))

## Issue
Deprecate obsolete, vulnerable or legacy packages [#2867](https://github.com/NuGet/Home/issues/2867)

## Problem
1. There are certain cases when a package author or NuGet repository Admin (nuget.org admin) should be able to let the package consumers know that a certain package (version) should no longer be used i.e. **deprecated**. As summarized in one of the comments on the issue, following are the most important cases for package deprecation:
 * `Vulnerable`: When the package version contains a security vulnerability and the author recommends not using the package and instead a newer patched version is recommended.
    * Authors should optionally be able to provide a CVE number
 * `Legacy`: When the package is no longer maintained by the author and author may have published another package (ID) instead.  
 * `Deprecated`: (catch misc reasons like performance, critical bugs, etc.)When a version of the package is deprecated by 
the author and the author recommends either not using the package+version or using a newer non-deprecated version.

  Today, this is partially possible by un-listing the package but there is no explicit feedback to the package consumers that a certain package version should no longer be used once its already in the project's list of packages (direct of full closure including transitive packages). 

2. In addition to the above, there is no mechanism for the package authors to suggest a new package or a new version to the package consumers where the issue (due to which the package was deprecated) has been fixed.

## Evidence
Being able to flag the security vulnerabilities in the packages is the primary reason behind taking up this feature. GitHub already flags the repos that depend on vulnerable packages. Today this is not enabled for NuGet packages as we do not have this feature. Similarly, npm also has the ability to flag vulnerable packages. 

## Solution
### Marking a package as vulnerable (Server)
### Flagging vulnerable packages used in a project (Client)