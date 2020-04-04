* Status: In Progress
* Author(s): Chris Gill (chgill), Joel Verhagen (jver)
* Issue: [Show dependent packages for a given package on the details page #4718](https://github.com/NuGet/NuGetGallery/issues/4718)
* Type: Feature

## Problem Background

**TL;DR:** 
* For consumers, this is a package trust indicator 
* For Authors, dependents can be a success metric/accolade
* Package consumers are more likely to have context around NuGet usage than GitHub usage

Package consumers want to be able see what packages are depending on a particular package of interest for a couple key reasons:
1. **Trust/quality indication** - Presumably, if popular/high-quality/many packages depend on a particular package, then that package is more likely to be high-quality itself.
2. **Author metrics/accolades** - Authors are often interested in seeing how many and what unique packages are directly depending on their package. Total number of downloads doesn't tell the full the story since transitive downloads, upgrades, and redownloads all count. Having a high-profile package depend on your package can also be a big deal!

_NuGet already shows GitHub Usage, what does showing dependent packages add?_

Not all NuGet package consumers use GitHub or are interested in GitHub usages. Those individuals are also less likely to be familiar with what repo stars mean, or if they do, they may have little reference for what constitutes a high or low number of stars. This lack of context significantly reduces the impact of GitHub usage as a trust indicator.

In contrast, popular packages may be more likely to be recognized by NuGet users since they may have seen them before in search results and total number of downloads is likely more straightforward to understand than GitHub stars. A user who is unsure how to interpret 15 stars on a repo is still likely to understand that 5 million downloads is a lot. Additionally, GitHub Usage can include repos that only transitively depend on the focus package, which can be somewhat misleading or unintuitive. 

## Who are the customers

NuGet package consumers (trust indicator) and package authors (metric) on both NuGet.org [and Visual Studio (TBD)]

## Requirements

**MVP:**

NuGet.org:
* Information available on package details page
* Total number of dependent packages.
* Top 10 most popular dependents packages (package ID).
* Dependents ordered by number of downloads.

**Next Steps:** (In Progress)

Visual Studio:
* Display top 10 most popular dependents in package details section.
* Order dependents by popularity.

NuGet.org
* Link to see all dependent packages sorted by popularity
* Package descriptions with package IDs
* Package verified check mark?
* Package owners?
* Filter package dependents by version being viewed?
* Filter search with `dependents:`

Other
* Public API to query for dependents of a package

## Goals

1. Give package consumers better tools to gauge package trustworthiness
2. Give authors more ways to gauge package success/growth

## Non-Goals

* Display complex information about TFMs and which version of a dependent depends on which version of the package
* Display transitive dependents
* Filter between pre-release and stable dependents

This should be simple to understand and use. It's not meant to be used as a complex dependency tracing tool.

## Solution

For clarity, I'll be describing the package whose details page is being viewed as the "focus package."

### Criteria for dependent package:

_Latest stable version_ depends on _any version_ of the focus package.

We are framing the appearance of a dependent package as an "endorsement" from that package. By using the focus package in their latest stable version, they are essentially endorsing the focus package. We only display a package as a dependent if the latest stable version depends on the focus package because a transition away from the focus package may indicate they are no longer willing to endorse it. 

### MVP Mock Ups

#### NuGet.org details page default/initial view

![image](https://user-images.githubusercontent.com/15097183/78413368-e9d3bc80-75cb-11ea-8dc4-1cbfe9bd101a.png)

* "Dependents" was chosen as the name of the dropdown because it's a well known term in package management. However, there is some concern that it may be easily confused with "Dependencies" or unclear to beginners.
* GitHub Usage and dependent packages are combined into a single dropdown because they have extremely similar purposes. Having separate dropdowns will also unnecessarily increase the space on the package details page taken up by various dependents of the focus package.

#### Dependents expanded

![image](https://user-images.githubusercontent.com/15097183/78414866-26ef7d00-75d3-11ea-9ff2-a4edc829c4f4.png)

* The tabbed structure prevents the content of the dropdown from getting very long and forcing the user to scroll to see everything. 
* Dependent packages are displayed by default because its more aligned with the NuGet platform and is likely a better/more intuitive trust indicator than GitHub Usage for the reasons stated in the problem background.
* Owners and verified check marks are left off to be able to release this more quickly (MVP).

## Future Work (TBD)

Visual Studio:
* Display top 10 most popular dependents in package details section.
* Order dependents by popularity.

NuGet.org:
* Link to see all dependent packages sorted by popularity
* Package descriptions with package IDs
* Package verified check mark?
* Package owners?
* Filter package dependents by version being viewed?
* Filter search with `dependents:`

Other:
* Public API to query for dependents of a package

### Future Mock Ups

#### Package Usage expanded (w/ description, owners, verified check, and link to full list)

![image](https://user-images.githubusercontent.com/15097183/78413414-29020d80-75cc-11ea-8b7d-f02d73bb3ea0.png)

#### Full list of dependent packages

**Note:** Dependent package IDs don't match previous image here, but they will in the real deal.

![image](https://user-images.githubusercontent.com/15097183/78312974-3787f080-750a-11ea-8f73-0dbfd4575636.png)

#### Visual Studio display

![image](https://user-images.githubusercontent.com/15097183/78415294-ac742c80-75d5-11ea-832a-9ed462f0059a.png)

## Open Questions

* What is the best way to surface this information in Visual Studio?
* Is our criteria for dependent package the most helpful/intuitive?
* Should we also enable a `dependents: <package ID>` search feature?

## Considerations

*TBD*

## References

*TBD*