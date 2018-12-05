* Status: **Incubation**
* Author(s): [Anand Gaurav](https://github.com/anangaur) ([@adgrv](https://twitter.com/adgrv))

## Issue
Deprecate obsolete, vulnerable or legacy packages [#2867](https://github.com/NuGet/Home/issues/2867)

## Problem
1. There are certain cases when a package author or NuGet repository Admin (nuget.org admin) should be able to let the package consumers know that a certain package (version) should no longer be used i.e. **deprecated**. As summarized in one of the comments on the issue, following are the most important cases for package deprecation:
 * `Vulnerable`: When the package version contains a security vulnerability and the author recommends not using the package and instead a newer patched version is recommended.
    * Authors should optionally be able to provide CVE number(s) and other vulnerability metrics. 
 * `Legacy`: When the package is no longer maintained by the author and author may have published another package (ID) instead.  
 * `Misc.`: (other reasons like performance, critical bugs, etc.) When a version of the package is deprecated by 
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

## Publisher experience
### NuGet.org
1. Package authors goes to the **Manage package** page and expands the **Deprecate Package** section. 

   > Note that all the package related management is on one page now.

   ![image](https://user-images.githubusercontent.com/14800916/49535977-532da900-f87a-11e8-9468-bb774439630b.png)

1. Is prompted to **select a reason**:
   
   > By default, **hide package** checkbox is checked i.e. upon `Save`, the package will be deprecated as well as hidden.

   ![image](https://user-images.githubusercontent.com/14800916/49536154-b586a980-f87a-11e8-9b2c-c853c38be51a.png)

1. Reason = **Vulnerable**. Select versions (multi-select)
   
   ![image](https://user-images.githubusercontent.com/14800916/49536256-ed8dec80-f87a-11e8-8efe-1bb790def53d.png)

   Provides the **optional details** like [CVE#](https://cve.mitre.org/), [CWE text](https://cwe.mitre.org/) and [CVSS score](https://www.first.org/cvss/specification-document#5-Qualitative-Severity-Rating-Scale)
   
   ![image](https://user-images.githubusercontent.com/14800916/49536486-7dcc3180-f87b-11e8-9844-b218978e469f.png)

1. Reason = **Legacy**. Applies to **all versions** of the package. Selects alternate package ID (optional) already available on nuget.org

   > There would be auto-complete and verification built-in for existing packages. If a non-existent package ID is entered, the save button will be disabled and error message printed.

   ![image](https://user-images.githubusercontent.com/14800916/49536760-2084b000-f87c-11e8-8b76-e6788c5b9005.png)

1. Reason = **Misc./other**: When package has some issues where a newer package version must be used. 
  
   ![image](https://user-images.githubusercontent.com/14800916/49536816-4b6f0400-f87c-11e8-89b3-ed7fae6478cb.png)

For now, providing a reason text from author is not enabled for any deprecation. We may choose to enable it in future. 

### CLI command experience:
```
// The package version should be deprecated because it has security vulnerabilities. By default hides too.
> dotnet nuget deprecate My.Demo.package * --vulnerable --cve CVE-2018-xxxx,CVE-2018-yyyy --cvss 2.3 --cwe xxx-vulnerability 

// The package version should be deprecated because it is legacy and no longer maintained
> dotnet nuget deprecate My.Demo.package --legacy --alternate My.Demo.Maintained.Package

// The package version should be deprecated because it has some issues that are fixed in later versions
> dotnet nuget deprecate My.Demo.package [2.0.0, 3.0.0)  

// Deprecate but do not hide
> dotnet nuget deprecate My.Demo.package 1.0.0,1.1.0 --vulnerable --cve CVE-2018-xxxx –no-hide

// remove deprecation
> dotnet nuget deprecate --undo My.Demo.package 1.0.0,1.1.0

// re-listing a hidden package
> dotnet nuget delete –-undo My.Demo.package 1.1.0

// push packages as un-listed
> dotnet nuget push My.Demo.package -k 4003d786-cc37-4004-bfdf-c4f3e8ef9b3a] --hide
```

The same command with all the above options should be present in nuget.exe as well.

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




