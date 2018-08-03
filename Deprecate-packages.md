* Status: Planned
* Author(s): [Anand Gaurav](https://github.com/anangaur) ([@adgrv](https://twitter.com/adgrv))

## Issue
Deprecate obsolete, vulnerable or legacy packages [#2867](https://github.com/NuGet/Home/issues/2867)

## Problem
1. There are certain cases when a package author or NuGet repository Admin (nuget.org admin) should be able to let the package consumers know that a certain package (version) should no longer be used i.e. **deprecated**. As summarized in one of the comments on the issue, following are the most important cases for package deprecation:
 * `Vulnerable`: When the package version contains a security vulnerability and the author recommends not using the package and instead a newer patched version is recommended.
    * Authors should optionally be able to provide a CVE number
 * `Legacy`: When the package is no longer maintained by the author and author may have published another package (ID) instead.  
 * `Deprecated`: (other reasons like performance, critical bugs, etc.)When a version of the package is deprecated by 
the author and the author recommends either not using the package+version or using a newer non-deprecated version.

  Today, this is partially possible by un-listing the package but there is no explicit feedback to the package consumers that a certain package version should no longer be used once its already in the project's list of packages (direct of full closure including transitive packages). 

2. In addition to the above, there is no mechanism for the package authors to suggest a new package or a new version to the package consumers where the issue (due to which the package was deprecated) has been fixed.

## Evidence
Being able to flag the security vulnerabilities in the packages is the primary reason behind taking up this feature. GitHub already flags the repos that depend on vulnerable packages. Today this is not enabled for NuGet packages as we do not have this feature. Similarly, npm also has the ability to flag vulnerable packages. 

## Solution
### Marking a package as vulnerable (Server)
Package publishers should be able to mark a given package (and version) as **deprecated** with one of the 3 reasons:
* Vulnerable
* Legacy
* Deprecated

and optionally provide additional information like CVE#(vulnerable packages), recommended package and version.

Detailed publisher experience storyboard can be found [here](#publisher-experience). The same experience should be possible for a nuget.org admin.

### Flagging vulnerable packages used in a project (Client)
Once a package has been deprecated, they are hidden in search results. If any deprecated package is already being used in a project, it should be:
1. Flagged during `restore`:
```
<<TBD>>
```
2. With `list --deprecated`:
```
<<TBD>>
```
3. On Visual Studio, this should be flagged on the Updates tab of the Package Manager UI:

<<TBD storyboard>>

## Publisher experience
### nuget.org
1. Package authors goes to the **Edit package** page:
  ![image](https://user-images.githubusercontent.com/14800916/43664918-6e9dac48-9723-11e8-853e-e537b815c7f5.png)

2. Checks **Unlist package**:
  ![image](https://user-images.githubusercontent.com/14800916/43665006-a0ea6330-9723-11e8-9c1c-fba03181e50f.png)

3. Is prompted to **select a reason**:
  ![image](https://user-images.githubusercontent.com/14800916/43665021-b46b4d98-9723-11e8-9e29-033acd2a4f51.png)

4. Reason = **Vulnerable** 
  ![image](https://user-images.githubusercontent.com/14800916/43665044-c4c2d968-9723-11e8-9b4a-8aed0fc8c82d.png)

5. Provides the **optional details**:
  ![image](https://user-images.githubusercontent.com/14800916/43665101-e7c90b58-9723-11e8-8da8-8fe9a9feafed.png)

6. Reason = **Legacy**
  ![image](https://user-images.githubusercontent.com/14800916/43665620-a90e4a02-9725-11e8-8e1f-08b756ec400c.png)

7. Reason = **Deprecated** (for any other reason):
  ![image](https://user-images.githubusercontent.com/14800916/43665123-fd930876-9723-11e8-8e2c-8ec521926617.png)

8. Upon save, the setting reflects the **unlisted status**:
  ![image](https://user-images.githubusercontent.com/14800916/43665144-14d220bc-9724-11e8-956a-b67c87923980.png)

### CLI command experience:
  ![image](https://user-images.githubusercontent.com/14800916/43665175-2bbaef02-9724-11e8-886a-00cd638b5a06.png)