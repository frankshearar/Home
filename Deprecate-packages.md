* Status: **Incubation**
* Author(s): [Anand Gaurav](https://github.com/anangaur) ([@adgrv](https://twitter.com/adgrv))

## Issue
Deprecate obsolete, vulnerable or legacy packages [#2867](https://github.com/NuGet/Home/issues/2867)

## Problem
1. There are certain cases when a package author should be able to let the package consumers know that a certain package (version) should no longer be used i.e. **deprecated**. As summarized in one of the comments on the issue, following are the most important cases for package deprecation:
 * `Vulnerable`: When the package version contains a security vulnerability and the author recommends not using the package and instead a newer patched version is recommended.
    * Authors should optionally be able to provide CVE number(s) and other vulnerability metrics. 
 * `Legacy`: When the package is no longer maintained by the author and author may have published another package (ID) instead.  
 * `Other`: (other reasons like performance, critical bugs, etc.) When a version of the package is deprecated by 
the author and the author recommends either not using the package+version or using a newer non-deprecated version.

Today, package authors can un-list a package but there is no explicit feedback to the package consumers that a certain package version should no longer be used once its already in the project's list of packages (direct or full closure including transitive packages). 

2. In addition to the above, there is no mechanism for the package authors to suggest a new package or a new version to the package consumers where the issue (due to which the package was deprecated) has been fixed.

## Evidence
Being able to flag the security vulnerabilities in the packages is the primary reason behind taking up this feature. GitHub already flags the repos that depend on vulnerable packages. Today this is not enabled for NuGet packages as we do not have this feature. Similarly, npm also has the ability to flag vulnerable packages. 

## Solution
### Marking a package as vulnerable (Server)
Package publishers should be able to mark a given package (and version) as **deprecated** with one of the 3 reasons:
* Vulnerable
* Legacy
* Other

and optionally provide additional information like CVE#(vulnerable packages), recommended package and version.

## Out of scope
* Ability to show consumers all vulnerable packages by scanning all CVEs - This will be a useful feature. However, out of scope for this feature.
* Ability to let package authors know if the package might be vulnerable - While we are exploring this option, but is out of scope for this feature.
* A command to audit all packages in a project/solution and produce vulnerability report - out of scope for this feature.

## Publisher experience
### NuGet.org
1. Package authors goes to the **Manage package** page and expands the **Deprecation** section. 

   > Note that all the package related management is on one page now.

   ![image](https://user-images.githubusercontent.com/14800916/49603057-05c73f80-f93f-11e8-9ef9-3f29a8862b0a.png)

1. Is prompted to **select version(s)**:

   > By default, the version from the package version page which led to this page, is selected. Authors can choose one or many versions.

   ![image](https://user-images.githubusercontent.com/14800916/50028628-2d548280-ffa5-11e8-8a18-b5eb8d5de7ef.png)

1. **Select reason(s)**:
   
   > By default, **hide package** checkbox is checked i.e. upon `Save`, the package will be deprecated as well as hidden.

   ![image](https://user-images.githubusercontent.com/14800916/50028711-6e4c9700-ffa5-11e8-81f3-ad6c62ae8996.png)

1. Reason = **Vulnerable**. Provides the **optional details** like [CVE#](https://cve.mitre.org/), [CWE text](https://cwe.mitre.org/), [CVSS score](https://www.first.org/cvss/specification-document#5-Qualitative-Severity-Rating-Scale) and any custom message for consumers.
   
   ![image](https://user-images.githubusercontent.com/14800916/50028785-a48a1680-ffa5-11e8-9c86-79cfb2079e8b.png)

1. Reason = **Legacy**. 
    
   > Warning if `All current versions` not selected. Authors can choose to ignore this warning and continue with a subset of available versions.
   
   ![image](https://user-images.githubusercontent.com/14800916/50028862-ddc28680-ffa5-11e8-92e3-bfe1636a30a6.png)

   > While selecting alternate recommended package, there would be auto-complete and verification built-in for existing packages. If a non-existent package ID is entered, the save button will be disabled and error message printed.

   ![image](https://user-images.githubusercontent.com/14800916/50028899-00549f80-ffa6-11e8-9c7e-c501b244a45a.png)
 
1. Reason = **Misc./other**: When package has some issues where a newer package version must be used. 
  
   ![image](https://user-images.githubusercontent.com/14800916/50028923-12ced900-ffa6-11e8-90f7-b62e78a1efd8.png)

#### Additional scenarios

* Selecting multiple reasons
   * If package author chooses both `legacy` and `vulnerable` reasons for the package versions to be deprecated, then both the `security details` and `alternate package` options open up for the author to provide.

* Selecting multiple versions with different deprecation states - i.e. wither some of the selected versions were already deprecated or all the versions were deprecated but due to different reasons.
  * The version list should show the deprecation and listing status in the drop-down:

    ![image](https://user-images.githubusercontent.com/14800916/50029106-abfdef80-ffa6-11e8-8320-2092bdba8c48.png)

  * If multiple versions with different deprecation states are selected (either some are deprecated and others are not OR all are deprecated with different reasons), an error is shown and `Save` is blocked:
     
    ![image](https://user-images.githubusercontent.com/14800916/50029206-f5e6d580-ffa6-11e8-8ad6-92b4e50a2d68.png)

  * If versions in the same deprecation state are selected, the deprecation reason is populated. The additional details like Security details or alternate package details are shown if they happen to be exactly same across selected versions, else the following info message is shown:
     
    `The selected versions have been deprecated with different details and hence cannot be shown here. If you want to modify additional details, please select the versions one by one.`

   * Authors can now either clear existing reasons or add more reasons to the already deprecated packages.  
   
### CLI command experience

There is no client command required for the MVP. 

## Consumer experience
### NuGet.org
> This experience is being iterated upon actively and hence may change a lot over coming days. 

1. Deprecated due to security vulnerability

   ![image](https://user-images.githubusercontent.com/14800916/49538131-91799700-f87f-11e8-83e6-4b31a10788e8.png)

   Package author should be able to navigate to deprecate settings by clicking on the **deprecated** link.
   
   ![image](https://user-images.githubusercontent.com/14800916/49538227-c980da00-f87f-11e8-8950-081b3db38e8f.png)

1. Deprecated due to legacy

   ![image](https://user-images.githubusercontent.com/14800916/49537883-000a2500-f87f-11e8-906e-bdfec3ef3d77.png)

1. Deprecated due to misc./other reasons

   ![image](https://user-images.githubusercontent.com/14800916/49538276-ea492f80-f87f-11e8-83ad-a60ce1e77122.png)

### Visual Studio

1. Flagged on the package listing

   ![image](https://user-images.githubusercontent.com/14800916/49548933-212d3e80-f89c-11e8-8b16-fbfb0fd2fa60.png)

1. Flagged during/after `restore`:

   ![image](https://user-images.githubusercontent.com/14800916/49548956-36a26880-f89c-11e8-9bc2-33a25bddf4e8.png)

1. Flagged during browse if the latest package is deprecated but not unlisted. Or if the deprecated package is selected in the drop-down before installing
   
   ![image](https://user-images.githubusercontent.com/14800916/50255818-55c6ed00-03a8-11e9-935f-2db8e1c3b354.png)

### CLI

1. Flagged on the package listing
The list command should be able to output deprecated packages with `--deprecated` option

```
// dotnet list command should output the deprecated packages with --deprecated option
> dotnet list package --deprecated
Run ‘dotnet audit’ 
The following sources were used:
   nuget.org - https://api.nuget.org/v3/index.json
   Local - C:\Play\NuGet\NuGetLocal

Project `ProjectB` uses the following deprecated packages
   [netcoreapp2.0]:
   Top-level Package      Resolved	Reason		Recommendation		
   > My.Demo.Package         2.0.0 	vulnerable (High Risk)	Use version 3.2.0 (potential breaking changes)
   > My.Legacy.Package	1.1.0	legacy 		Use My.Awesome.Package 3.0.0 (latest version

```
> To see all packages including transitive package, additional option `--include-transitive` can be used. 

1. Flagged during/after `restore`: 
During restore, NuGet should warn with the same text as shown in the VS UI.

### Restore warnings
* By default, all the warnings should have a warning ID (NUxxxx) and the text. And these warnings can either be suppressed or elevated as an error like other NuGet warnings.
* Each of the following need to have separate warning IDs and text
  * Warning when deprecated due to **vulnerability**
    * Low Risk
      ```
      NUxxxx: <text TBD>
      ```
    * Moderate Risk
      ```
      NUxxxx: <text TBD>
      ```
    * High Risk
      ```
      NUxxxx: <text TBD>
      ```
    * Critical Risk
      ```
      NUxxxx: <text TBD>
      ```
  * Warning when deprecated due to **legacy**
    ```
    NUxxxx: <text TBD>
    ```
  * Warning when deprecated due to **misc./legacy**
    ```
    NUxxxx: <text TBD>
    ```
* Restore warnings should show upon every **full restore** run, except when it NoOps i.e. when all packages were already restored and no further action was needed. This implies that even when the packages were already in Global packages folder, the deprecation warning should be shown.
* Search/Browse and List experiences should be able to always show the deprecation information from multiple sources if multiple repositories referenced have same/different deprecation states. **TBD**: the storyboard/screen.
* **Out of scope**:
  * Showing deprecation warning if the package is served from a repository that does not support deprecation.
  * Ability to show deprecation warnings for packages served from file/UNC based repositories.
  * NuGet `restore` showing warning consistently when the same package is hosted on more than 1 repository where one repository has deprecated the package but others have not. This is because `restore` picks up the package from the source that responds the fastest. **Note**: This information should be shown on package listing (VS package manager or CLI) i.e. even if one of the sources has the deprecation information. 
  * Showing deprecation warning if package is sourced from a repository that does not have deprecation information even if it hosts packages from another repository, supporting deprecation functionality, as upstream or offline copy.