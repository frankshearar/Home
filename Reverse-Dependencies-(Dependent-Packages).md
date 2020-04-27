
* Status: In Progress
* Author(s): Chris Gill (chgill), David Cueva Cortez (t-dacue)
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

![image](https://user-images.githubusercontent.com/15097183/80406423-e8b74780-8878-11ea-89ec-c15effd42bc9.png)

* GitHub and NuGet package dependents are combined into a single dropdown because they have extremely similar purposes. Having separate dropdowns will also unnecessarily increase the space on the package details page taken up by various dependents of the focus package.

#### Dependents expanded (package dependents tab)

![image](https://user-images.githubusercontent.com/15097183/80407810-0d142380-887b-11ea-8417-e489898cdb54.png)

* Owners and link to full list of dependent packages are left off to be able to release this more quickly (MVP).

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

#### Package Usage expanded w/owners and link to full list

![image](https://user-images.githubusercontent.com/15097183/80407434-7a738480-887a-11ea-81c7-8fa9707ac11d.png)


#### Full list of dependent packages

**Note:** Dependent package IDs don't match previous image here, but they will in the real deal.

![image](https://user-images.githubusercontent.com/15097183/78312974-3787f080-750a-11ea-8f73-0dbfd4575636.png)

#### Visual Studio display 

![image](https://user-images.githubusercontent.com/15097183/79286116-6ee48e80-7e74-11ea-92dd-08c4a31a5bb4.png)

## Open Questions

* What is the best way to surface this information in Visual Studio?
* Is our criteria for dependent package the most helpful/intuitive?
* Should we also enable a `dependents: <package ID>` search feature?

## Considerations

*TBD*