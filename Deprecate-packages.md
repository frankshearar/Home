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

1. Is prompted to **select reason(s)**:
   
   > By default, **hide package** checkbox is checked i.e. upon `Save`, the package will be deprecated as well as hidden.

   ![image](https://user-images.githubusercontent.com/14800916/49976030-3d148e00-fef5-11e8-8601-d70e7487d4f9.png)

1. Reason = **Vulnerable**. Select versions (multi-select)
   
   ![image](https://user-images.githubusercontent.com/14800916/49976063-5ddce380-fef5-11e8-8be2-a47be2979159.png)

   Provides the **optional details** like [CVE#](https://cve.mitre.org/), [CWE text](https://cwe.mitre.org/), [CVSS score](https://www.first.org/cvss/specification-document#5-Qualitative-Severity-Rating-Scale) and any custom message for consumers.
   
   ![image](https://user-images.githubusercontent.com/14800916/49976110-7d740c00-fef5-11e8-9426-eb50bb9d2624.png)

1. Reason = **Legacy**. Applies to **all versions** of the package. Selects alternate package ID (optional) already available on nuget.org

   > While selecting alternate recommended package, there would be auto-complete and verification built-in for existing packages. If a non-existent package ID is entered, the save button will be disabled and error message printed.

   > `All current versions` will be automatically selected. This can be overridden though.

   ![image](https://user-images.githubusercontent.com/14800916/49976179-c0ce7a80-fef5-11e8-9211-d647deaaeb25.png)
 
1. Reason = **Misc./other**: When package has some issues where a newer package version must be used. 
  
   ![image](https://user-images.githubusercontent.com/14800916/49976248-fe330800-fef5-11e8-8fd7-5d1d7cd6a0a3.png)

#### Additional scenarios

* Selecting multiple reasons
   * If package author chooses both `legacy` and `vulnerable` reasons for the package versions to be deprecated, then both the `security details` and `alternate package` options open up for the author to provide.

* Selecting multiple versions with different deprecation states
  * If you select multiple versions and some of them were either already deprecated, then the following should happen:
     * The already deprecated versions should be shown in red with deprecation and reason texts appended to them:
       ![image](https://user-images.githubusercontent.com/14800916/49976830-7e5a6d00-fef8-11e8-8199-48b66b72fa39.png)
     * If multiple versions with different deprecation states are selected (either some are deprecated and others are not OR all are deprecated with different reasons), an error is shown and `Save` is blocked:
       ![image](https://user-images.githubusercontent.com/14800916/49976917-e90ba880-fef8-11e8-9473-469aa0de8009.png)
     * If the selected versions are in already deprecated with a single reason (same deprecated state), and another reason is being applied to it, this is just additive. The information is shown as a warning as follows:       
       ![image](https://user-images.githubusercontent.com/14800916/49977080-b57d4e00-fef9-11e8-8d87-141965e95152.png)
     
### CLI command experience

There is no client command required for the MVP. 

## Consumer experience
### NuGet.org

> The package author message may not be available immediately. We may enable it in future.

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

### Warnings
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
