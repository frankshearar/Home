* Status: **Reviewing**
* Author(s): [Anand Gaurav](https://github.com/anangaur) ([@adgrv](https://twitter.com/adgrv)), [Xavier Decoster](https://github.com/xavierdecoster) ([@xavierdecoster](https://twitter.com/xavierdecoster))

## Issue

Flag vulnerable packages [#8087](https://github.com/NuGet/Home/issues/8087)

## Problem

Today developers take dependency on a set of packages directly or indirectly (through transitive dependencies) and have no way to understand if any of their dependencies bring in any known vulnerabilities.

## Goals

This is a security feature aiming to fill a gap in the .NET package management ecosystem known as *package vulnerability auditing*.

NuGet should be able to flag vulnerable packages used in projects/solutions.

The ultimate goal is to *reduce exposure to known vulnerabilities* in NuGet packages, and *increase adoption rate of fixes* for those known vulnerabilities.

## Non-Goals

* At this point, NuGet.org won't have a way for users to input vulnerability data into the system. All vulnerability data will be sourced from external systems. (no self-reporting)
* This feature is an enhancement to the NuGet v3 protocol. We won't surface this in earlier versions of the NuGet protocol.
* `NuGet.Server` is out-of-scope for this feature at this point.

## Scope

There are multiple scenarios for flagging and reporting vulnerabilities. This feature will be implemented in several stages which will enable a subset of those as discussed below.

### Stage 1 - MVP

For the [MVP](https://en.wikipedia.org/wiki/Minimum_viable_product) of this feature, we'll focus on:

* Enabling ***consumers** to be informed about known vulnerabilities before installation* of a package.
* Enabling ***authors** to be informed about known vulnerabilities in their packages*.
* Leveraging **existing, curated, and publicly available known vulnerabilities** affecting NuGet packages.

This should help prevent *new* installs of packages that are known to be vulnerable, and trigger authors into providing a fix through a new package version on nuget.org.

NuGet will not support self-reporting of vulnerabilities at this time and instead rely on an existing data provider of curated vulnerability information. For the MVP of this feature, all vulnerability data will be sourced from external systems. The initial focus is on leveraging [GitHub's Security Workflow](https://github.com/features/security) and integration with [GitHub's GraphQL API](https://developer.github.com/v4/). This allows us to build an initial end-to-end experience without having to deal with the complexities of curating vulnerability reports.

#### ✔️ In scope (Stage 1)

* For CLI:
  * A CLI command to **list vulnerabilities** in a project/solution project graph

* For Visual Studio:
  * **Display vulnerability information** in the package graph on `Installed` tab of Package Manager Dialog (PMUI)
  * **Show transitive packages for `PackageReference` type projects** in PMUI

* For nuget.org:
  * **Automatically ingest and refresh NuGet package vulnerability data from GitHub**
  * **Surface package vulnerability information on-demand**:
    * as part of package metadata in `v3` registration blobs (`JSON`)
    * by showing vulnerability information to package owners (only!) on package details page and version history
  * Basic (e-mail) notification to notify package publishers for any new vulnerability detected in one or more of their packages

#### ❌ Out of scope (Stage 1)

Some enhancements to this feature are out of scope for the initial implementation of this feature because there are several unknowns still to be clarified. See also the *Open Questions* section below.

* Enable `restore` to Inform or Warn (see Stage 2)
* Expose vulnerability metadata in nuget.org *Search* service (not required for MVP)
* Display vulnerability information for *transitive packages* in PMUI (additional UX complexity)
* Expose vulnerability information *publicly* on nuget.org's package details page and version history (disclosure policies TBD)

### Stage 2

Stage 2 of this feature will focus on *highlighting known vulnerabilities when the package is already installed*. This involves introducing new validations at package `restore` time, and new ways of dealing with them.

#### ✔️ In scope (Stage 2)

* For CLI:
  * Output warnings or errors when running `dotnet add package` (which actually does a *restore*)

* For Visual Studio:
  * Inform consumers about known vulnerabilities at time of installation or restore:
    * Warn (⚠️) for `critical` and `high` impact vulnerabilities (minimal info + command/way to uncover detailed information)
    * Info (ℹ️) for lower impact vulnerabilities (minimal info + command/way to uncover detailed information)
  * Display vulnerability information for *transitive packages* in PMUI
  * New settings to enable/disable warnings for the various impact levels of vulnerabilities

* For nuget.org:
  * Show vulnerability information in packages to their publishers on nuget.org
    * Show the vulnerability info on the package page and version history

#### ❓ Open questions (Stage 2)

* For `restore`, should it be opt-in or opt-out (preferred for security features)?
* For `dotnet add package`, define behavior (info, warn, error?) when encountering packages known to be vulnerable.
* Need for another command (e.g. `dotnet nuget audit`) to create an audit report for deprecated, vulnerable and may be outdated packages. And a way to fix the vulnerabilities with a single command/click.
* Disclosure policies if and when we start consuming non-disclosed vulnerability data? (e.g. under review by package author) - GH API currently doesn't surface these

| Feature | State |
|:--- |:--- |
| A way to specify another source for vulnerability information | Backlog |
| `why` command to understand why a package is shown as a dependency for my project | Backlog |
| Ability to report a vulnerability on nuget.org | Discuss |
| An API that lists all vulnerable nuget.org packages with the vulnerability information | Discuss |
| Show vulnerability information as part of search results | Discuss |
| Show vulnerability information publicly for each package like deprecation info on the package details | Discuss |

Legend:

* *Backlog: Follow-up feature to be implemented after the current one is implemented*
* *Discuss: Feature that needs further discussion*

## Solution (Stage 1)

### Protocol

To be able to show vulnerability information for phase 1 in PMUI and CLI clients, we need to update the protocol.

To this end, the following changes will happen to the package registration JSON blobs (to each *version-specific* blob, not the *index.json*):

| Name | Type | Required | Description |
|---|---|---|---|
| vulnerabilities | Array of Objects | No | Describes the vulnerabilities information associated with a package |
| vulnerabilities[x].severity | String | Yes | "Low", "Moderate", "High", or "Critical" |
| vulnerabilities[x].advisoryUrl | String | Yes | URL to discover additional information about the security advisory for this vulnerability |

#### Forward Compatibility

Clients should just not display the severity if they don't understand the value that is returned. The `severity` element values can be treated *case-insensitive*.

### List Vulnerabilities using a CLI command

[See client spec for `dotnet list package --vulnerable`.](https://github.com/NuGet/Home/wiki/dotnet-list-package---vulnerable)

### Display Vulnerabilities in Visual Studio PMUI

* `Installed` tab shows a warning icon that leads to vulnerable package, when the package has **High** or **Critical** vulnerabilities
* `Installed` tab also shows transitive packages as an accordion - collapsed, by default. It has a warning icon too when any of the transitive packages have vulnerabilities (**High** or **Critical**)

   ![image](https://user-images.githubusercontent.com/14800916/65348883-057af680-db97-11e9-8a2a-b02be76c9668.png)

   Showing vulnerability in transitive package:
   ![image](https://user-images.githubusercontent.com/14800916/65349163-9f42a380-db97-11e9-8de5-c81613e47d5d.png)

> **Note**: Irrespective of the criticality of the vulnerability, the details pane will always show the vulnerability information - with warning icon for Critical/High vulnerabilities and info for others. Just that Installed tab won't warn in those cases.
