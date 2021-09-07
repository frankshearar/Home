
* Status: In Review
* Authors: Chris Gill (chgill), David Cueva Cortez (t-dacue)
* Issue: [Show dependent packages for a given package on the details page #4718](https://github.com/NuGet/NuGetGallery/issues/4718)
* Type: Feature

## Table of Contents

* [Problem background](#problem-background)
* [Who are the customers](#who-are-the-customers)
* [Requirements](#requirements)
* [Goals](#goals)
* [Non-goals](#non-goals)
* [Solution](#solution)
* [Future Work](#future-work)
* [FAQ](#faq)
* [Open questions](#open-questions)
* [Considerations](#considerations)

## Problem background

**TL;DR:** 
* For consumers, this is a package trust indicator.
* For authors, dependents can be a success metric/accolade.
* Package consumers are more likely to have context around NuGet package usage/dependents than GitHub usage.


Customers want to see what packages are depending on a given package of interest for a couple key reasons:
1. **Trust/quality indication** - Presumably, if popular/high-quality/many packages depend on a particular package, then that package is more likely to be high-quality itself.
2. **Author metrics/accolades** - Authors are often interested in seeing how many and what unique packages are directly depending on their package. Total number of downloads doesn't tell the full story since transitive downloads, updates, and reinstalls all count. Having a high-profile package depend on your package can also be a big deal!

_NuGet already shows GitHub Usage, what does showing dependent packages add?_

Not all NuGet package consumers use GitHub or are interested in GitHub usages. Those individuals are also less likely to be familiar with what repo stars mean, or if they do, they may have little reference for what constitutes a high or low number of stars. This lack of context significantly reduces the impact of GitHub usage as a trust indicator.

In contrast, popular packages may be more likely to be recognized by NuGet users since they may have seen them before in search results and total number of downloads may be more straightforward to understand than GitHub stars. A user who is unsure how to interpret 2K stars on a repo is still likely to understand that 5 million downloads is a lot. Additionally, GitHub Usage can include repos that only transitively depend on the focus package, which can be somewhat misleading or unintuitive. 

## Who are the customers

NuGet package consumers (trust indicator) and package authors (metric) on NuGet.org.

## Requirements

**Initial release:**

NuGet.org:
* Table of dependent packages and repos easily accessible on package details page.
* Total number of dependent packages and repos displayed on sidebar.
* Shown dependents ordered by number of downloads/stars.

**Next steps:**

NuGet.org:
* Link to see all dependent packages sorted by popularity
* Package owners
* Filter search with `dependents:` (TBD)

Visual Studio:
* Display top N popular dependent packages in VS

Other
* Public API to query for dependents of a package

## Goals

1. Give package consumers better tools to gauge package trustworthiness.
2. Give authors more ways to gauge package success/growth.

## Non-goals

* All work listed under [Future work](#future-work) is out of scope for initial release.
* Display complex information about TFMs and which version of a dependent depends on which version of the package.
* Display transitive dependents.
* Filter between pre-release and stable dependents.
* Allow custom sorting on the package details page.

This should be quick to scan and simple to understand. It will not be a complex dependency tracing tool.

## Solution

For clarity, I'll be describing the package whose details page is being viewed as the "focus package." In the following examples, the focus package will be Newtonsoft.Json.

### Criteria for dependent package:

_Latest stable version_ depends on _any version_ of the focus package.

This means if a package depended on the focus package in version 2.0.0 but moved off of it for version 2.1.0, then it will no longer be displayed as a dependent. If a package is depending on version 1.0.0 of the focus package, but the latest version of the focus package is 1.1.0, it will still be displayed as a dependent regardless of which version of the focus package is currently being viewed.

We are framing the appearance of a dependent package as an "endorsement" from that package. By using the focus package in their latest stable version, a package is essentially endorsing the focus package. We only display a package as a dependent if the latest stable version depends on the focus package because a transition away from the focus package may indicate they are no longer willing to endorse it. 

For more details on this, please see the [FAQ section](#faq).

### Mock ups

#### NuGet.org details page default/initial view

![image](https://user-images.githubusercontent.com/15097183/80406423-e8b74780-8878-11ea-89ec-c15effd42bc9.png)

* GitHub and NuGet package dependents are combined into a single "Used By" dropdown because they have extremely similar purposes. Having separate dropdowns will also unnecessarily increase the space on the package details page taken up by various dependents of the focus package.
* "Used By" chosen for the title of the dependents drop-down because feedback indicated that the term "dependent" may be easily confused with "dependency." "Usage" was also considered to play off "GitHub Usage" but was decided against because it may create the expectation of usage examples rather than who uses the package.
* "Used By" section on the sidebar makes dependent information more easily scannable and further hints at the information contained in the "Used By" dropdown. Clicking the links in on the sidebar will open the dropdown and scroll to the appropriate section. Putting the dependents links in the "Info" section risks overcrowding it and presents a challenge for clear naming.

#### "Used By" expanded

![image](https://user-images.githubusercontent.com/15097183/80407810-0d142380-887b-11ea-8417-e489898cdb54.png)

* Owners and link to full list of dependent packages are left off for the initial release.
* Tables reduced to top 5 (as opposed to top 10) dependent repos/packages so that the expansion isn't too big. We want to avoid the need for excessive scrolling and not push down other useful content too far. Additionally, top dependents give the most useful information to consumers and this design makes that information as easy to scan as possible.
* "How do I use this?" link at the bottom links to documentation describing NuGet's recommended process for evaluating package trust, as well as describing how our dependents calculation/display works. Link goes to here: [https://docs.microsoft.com/en-us/nuget/consume-packages/finding-and-choosing-packages#evaluating-packages](https://docs.microsoft.com/en-us/nuget/consume-packages/finding-and-choosing-packages#evaluating-packages)
* Verified check marks lend an additional dimension of trust evaluation.

## Future work

NuGet.org:
* Link to see all dependent packages sorted by popularity
* Package owners
* Filter search with `dependents:` (TBD)

Visual Studio:
* Display top N popular dependent packages in VS

Other:
* Public API to query for dependents of a package

### Future mock-ups

#### Package Usage expanded w/owners and link to full list

![image](https://user-images.githubusercontent.com/15097183/80410017-b4df2080-887e-11ea-94bf-d5afda8bc442.png)

* Owners info adds another potential trust indicator if the owner is recognizable (Microsoft, dotnet foundation, etc.).
* "See all" link will open a new tab that displays a [view of all dependent packages](#full-list-of-dependent-packages).

#### Full list of dependent packages

**Note:** Dependent package IDs don't match previous image here, but they will in the real deal.

![image](https://user-images.githubusercontent.com/15097183/78312974-3787f080-750a-11ea-8f73-0dbfd4575636.png)

* **Caveat:** While the full list of dependent packages is surfaced using the NuGet.org search page UI, queries entered in the search bar will initiate a regular search. For example, searching for "Contoso" will bring search results from all of NuGet.org, not just from within the dependents of the focus package. To search for a specific package among the dependents, a search filter would be required which is being considered but is not currently planned.

#### Visual Studio display 

![image](https://user-images.githubusercontent.com/15097183/79286116-6ee48e80-7e74-11ea-92dd-08c4a31a5bb4.png)

* ******Note:** Very tentative design. "Dependent packages" is very unlikely to be the final wording due to concerns of confusion with "dependencies." Will likely require a separate research and design effort down the road.

## FAQ

**Q: Why does the "latest stable version" of a package have to depend on the focus package to be displayed as a dependent? Why not any version or the latest including pre-release?**

A: Dependent packages are framed as endorsements. If a package is no longer depending on the focus package in its latest stable version, it is as if the focus package has lost that endorsement (perhaps in favor of another packages). Pre-release dependencies aren't considered a shift in endorsement in either direction as pre-releases maybe be used for experimenting with new dependencies that perhaps haven't been tested sufficiently to be considered stable. It is difficult to discern the a package author's rationale for moving toward or away from a dependency, so only displaying the latest stable version can be considered relying on an author's current best judgement.

**Q: Why "Used By" for the dropdown title as opposed to "Dependents?"**

A: Customer feedback has indicated that the term "dependents" may be easily confused with "dependencies" which is a core concept of in package management. Having the terms right next to each other may add an additional layer of confusion. Other terms such as "Usage," which echoes the existing "GitHub Usage" title, were considered. However, "Usage" could be interpreted as examples of how to use a package rather than examples of where it is being used.

**Q: Will the display of dependents be specific to the version of the focus package I'm viewing? (If I am viewing Newtonsoft.Json 2.0.0, will the dependents tables only show packages in which the latest stable version of those packages are depending on the Newtonsoft.Json 2.0.0?)**

A: No, GitHub Usage does not currently behave that way, and neither will the dependent packages display. While we understand version specificity may aid in evaluating a specific version of a package, the concerns are as follows:
* Dependents may be very fragmented so few prominent dependents may be displayed for any single version. This may be especially the case for focus packages that release very frequently. For consumers, this may make this dependents information a weaker trust indicator for a package as a whole and somewhat discourages frequent releases.
* For the authors of the focus package, it makes using dependents as a success metric harder to track as they would have to click through every version to get a full picture.
* Dependency version isn't always a conscious choice on the part of a package author. Other factors may include wildcards or compatibility requirements. In this case, version specific displays may be misleading.
* Technical complexity for implementation is higher for potentially little ROI.

**Q: What happens with unlisted or deprecated dependent packages?**

A: Unlisted and deprecated packages cannot be displayed as dependent packages.

**Q: Can .NET tools be package dependents?**

A: No - it is a consideration for the future, but there are no plans for it at this time.

**Q: Are there currently plans for a "see all" view for the dependent GitHub repos?**

A: It is a consideration for the future, but there are no plans for it at this time.

## Open Questions

* What is the best way to surface this information in Visual Studio?
* Is our criteria for dependent package the most helpful/intuitive?
* Should we also enable a `dependents: <package ID>` search feature?
* Should we have feature parity for GitHub dependent repositories? (display of all dependent repos, surfacing in VS)

## Considerations

### Alternative mock-ups

#### Alternative sidebar placement and naming for dependent repos/packages

![image](https://user-images.githubusercontent.com/15097183/80410651-cd036f80-887f-11ea-95b0-36be89257ce6.png)


#### Alternative "Used By" section design

![image](https://user-images.githubusercontent.com/15097183/80410395-5a928f80-887f-11ea-8907-9c7c24c4964c.png)

* While this design allows for more dependents to be shown (top 10 for both packages and repos), it places GitHub dependent information a click further away and presents some accessibility issues.
