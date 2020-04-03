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

In contrast, popular packages may be more likely to be recognized by NuGet users since they may have seen them before in search results and total number of downloads is likely more straightforward to understand than GitHub stars. A user who is unsure how to interpret 15 stars on a repo is still likely to understand that 5 million downloads is a lot.

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

![image](https://user-images.githubusercontent.com/15097183/78312126-a44dbb80-7507-11ea-8304-4be8c5c4aad7.png)

* The "Package Usage" tab will contain the dependent packages information. "Package Usage" was chosen over "Dependents" or "Dependent Packages" to echo "GitHub Usage" since many package consumers will already be familiar with what that entails. Some preliminary feedback was also received that "Dependents" could be confused with the existing "Dependencies" tab.
* "Package Usage" will be a tab separate from "GitHub Usage" for a few key reasons:
    1. Combining the info under one tab might make the expansion annoyingly long.
    2. Combining the info under one tab would reduce the relative important of each one. It's not clear yet which 
       customers prefer to use.
    3. Total number of dependent packages can be displayed prominently as a trust indicator without even having to open 
       it.

#### Package Usage expanded

![image](https://user-images.githubusercontent.com/15097183/78312497-db709c80-7508-11ea-9ccb-c63263798de0.png)

* Similar UI to GitHub Usage display since it's a very similar feature. This table format also makes the top 10 dependents and their respective download counts very clear and prominent.

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

![image](https://user-images.githubusercontent.com/15097183/78312868-f1cb2800-7509-11ea-9dd0-99182c6c8b6c.png)

#### Full list of dependent packages

**Note:** Dependent package IDs don't match previous image here, but they will in the real deal.

![image](https://user-images.githubusercontent.com/15097183/78312974-3787f080-750a-11ea-8f73-0dbfd4575636.png)

#### Visual Studio display

**TBD**

## Open Questions

* What is the best way to surface this information in Visual Studio?
* Is our criteria for dependent package the most helpful/intuitive?
* Should we also enable a `dependents: <package ID>` search feature?

## Considerations

*TBD*

## References

*TBD&