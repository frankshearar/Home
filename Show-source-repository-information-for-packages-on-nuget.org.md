## Issue
Users should be able to see package repository metadata [#4941](https://github.com/NuGet/NuGetGallery/issues/4941)

## Problem
Look at the issue description for the background and the problem details. In short, since mid 2016, the NuGet package manifest (.nuspec) has supported the <repository> XML element. Earlier this year - 2018, we documented the `repository` property. But there is no clarity on the purpose of this repository, how this should be lit up on the NuGet Gallery, VS client or the format in which this should specified in the nuspec.

## Who is the customer?
* All the NuGet package publishers who are using this field in a certain way or are looking for recommendations on how to use this functionality
* All package consumers who would like to know the source code repository to make certain decisions. For example, developers look at the repository to understand how active the project is or if the package is open source (so that it can be serviced by them if required), etc. 

## Evidence
* A total of 35,231 packages have been published with <repository> metadata.
* There are 6,539 distinct package IDs that use repository metadata.

Look at the **Usage on nuget.org** section of the [linked issue](https://github.com/NuGet/NuGetGallery/issues/4941) for more details.

## Solution
### Purpose

Package publishers are often confused on the purpose of `projectUrl` vs. `repositoryUrl`. The purpose of `projectUrl` is to state the product/offering site. On the other hand the purpose of `repositoryUrl` is to state a machine readable Url to the repository. Note that while the `projectUrl` is a web Url, `repositoryUrl` is not.

E.g. for NuGet package `NuGet.Client` , `projectUrl` is https://nuget.org and `repositoryUrl` is https://github.com/NuGet/NuGet.Client.git

For many packages, both the `projectUrl` and `repositoryUrl` may point to the same repository (eg. on GitHub). However, one should note the Url format for these metadata properties (web vs. not).

Repository information is very useful to the package consumers as they tend to look for repository Url to understand if the project is open source and active to understand if their issues (future) are likely to be serviced or if they can contribute towards fixing some of them. Similary looking at the count of open issues and the velocity of fixes gives them important information to determine whether they should take a dependency on a given package or not. nuget.org plans to expose these information based on repository information to help consumers make this decision faster but is out of scope for this spec document.

### Authoring 
The repository information needs to be present in nuspec file. E.g:
```
<repository type="git" url="https://github.com/NuGet/NuGet.Client.git" />
```
* `type`: Specific the type of repository like `git`, `svn`, etc
* `url`: This should be a publicly available url that can be invoked directly by a version control software. It should not be an html page. This is meant for the machine. For linking to project page, use the `projectUrl` field.
* `branch`: *(optional)* additional metadata meant for `git` repositories
* `commit`: *(optional)* additional metadata meant for `git` repositories

### Validations
There should be validations at the time of `pack`, `push`|`update` and the error/warning should be shown to the user with the link to use the field appropriately. The validations should check for the following:
1. If the URL specified is not public URL.
  * `pack`: No check.
  * `push`|`upload`: Warning (nuget.org)
2. If the URL is not in the right format. (We should start with `git` URLs and add validations for other types later) 
  * `pack`: Warning
  * `push`|`upload`: Warning (nuget.org)
3. If the URL is not https
  * `pack`: No check.
  * `push`|`upload`: Warning (nuget.org) 
4. If the URL points to an inappropriate or malicious site
  * Validation (1) should be to mitigate to quite an extent. 
  * In future, we should plan to do content verification scan not just for the nuspec but for all the contents of the package.

### Display on nuget.org (short-term)
Since this metadata is already being used in many packages, we would like to go one step at a time and start showing this information on nuget.org: 

![](https://user-images.githubusercontent.com/2696087/42461816-4f1b81dc-8356-11e8-8556-b68b4c70eeca.png)

* We should show the provider icon (wherever possible e.g. GitHub) when the provider is known from the url
* We should show the repository type icon, if we cannot derive the provider but the repository type is a known one (e.g. git).
* For existing repository Urls that are not in the right format i.e. they point to the http/https (web) urls instead of the repository URL (eg. https://github.com/NuGet/NuGet.Client.git), we should still point to the web/http(s) urls.

### Long term ideas on the display and usage 
Out-of-scope for this spec. However, some of the ideas include:
* Show repository info as a separate section with links to file issues, show number of active issues and last activity - all of these are vital information that consumers would like to see for a given package they want to consume.
* Compute the quality metirc based on issues fixed vs. open issues on the repository
* Use this to switch from PackageReference to ProjectReference by automatically cloning the repo as a project.