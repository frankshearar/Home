# High Level Goals

Currently, the NuGet team is in the process of moving our package search back-end on nuget.org from a hand-rolled Lucene-based service to Azure Search. The goals of this effort are:

1. Improve search relevancy.
1. Improve time it takes for a new package to be indexed.
1. Reduce maintenance costs.
1. Improve agility for future search-based features.

The "epic" (high level work item) for this effort is [NuGet/NuGetGallery#6371](https://github.com/NuGet/NuGetGallery/issues/6371) but you can [search for "Azure Search" issues](https://github.com/NuGet/NuGetGallery/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+azure+search) and find all the little pieces that we are working on.

If you find specific search queries that don't work well today, please chime in on [NuGet/NuGetGallery#4124](https://github.com/NuGet/NuGetGallery/issues/4124). We're using these reports and an initial smoke test to validate our new implementation.

# Status

We've implemented most of the back-end and have package indexing to Azure Search running in parallel with the legacy search system. The nuget.org service index (https://api.nuget.org/v3/index.json) still points to the legacy system while we're working out the kinks.

We plan on introducing the new implementation to users along with a preview survey, A/B testing, and phased rollout to our various search integration points. This is the rough order of things we're planning:

1. [x] Implement the back-end and run the new stuff in parallel.
1. [ ] Play with the new search relevancy internally using a side-by-side comparison page.
1. [ ] Show the comparison page to a small set of external partners and gather feedback.
1. [ ] Tweet/announce the comparison page to the community and gather feedback.
1. [ ] Integrate nuget.org search with Azure Search using an A/B testing approach.
1. [ ] Move all of nuget.org search traffic to Azure Search.
1. [ ] Point V2 search to Azure Search.
1. [ ] Point V3 search to Azure Search.
1. [ ] Point nuget.org V2 hijack to Azure Search.

# Try it now!

You can try the new search service out in Visual Studio by using a preview service index that is exactly like the main one but swaps out the search implementations. This means you can install and restore packages as usual but search queries (like in the Browse tab in Visual Studio NuGet Package Manager) will go the new search implementation.

Use the following V3 package source URL instead of the default one to try it out:
```
https://apidev.nugettest.org/v3-index/azsprod-index.json
```

To do this in Visual Studio:

1. Go to the Tools > Options > NuGet Package Manager > Package Sources.
1. Add a new package source.
1. Set **Source:** to `https://apidev.nugettest.org/v3-index/azsprod-index.json`.
1. Disable the existing nuget.org source. This is not strictly necessary but will reduce wasted HTTP calls during restore.

![image](https://user-images.githubusercontent.com/94054/58720374-d5f57380-8386-11e9-8ae3-98bdcab0cca1.png)

# Feedback

If you find search queries on the new search service that you think need some work or if you have any other questions/feedback about our search redesign effort, please send them to support@nuget.org with the subject line **Search Feedback**.