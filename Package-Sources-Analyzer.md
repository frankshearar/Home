# NuGet Package Sources Analyzer

## Problem
Customers often complain of performance issues with retrieving packages in search, installing and updating packages. Often when we analyze these individual issues, we find that in a large percentage of cases the issues users are seeing are due to the following. Package Sources here refer to:- File shares, NuGet.Server's, 3rd party repositories and NuGet.org.

  * NuGet package sources response times are slow
  * Nuget package sources are behind a corporate firewall
  * User is offline
  * The backing file shares of NuGet.org are either not optimized or very large.

We want to proactively inform of the users of these issues, so they can take corrective action and/or give them an option to temporarily disable certain sources for a period of time of the session.

## Who is the customer?
Any customer who has a slow network, works offline and online, has NuGet.Server installations and have package sources behind corporate firewalls.

## Evidence
TBD

## Solution
_Detailed explanation of the solution. The more pictures/code snippets based on the feature the merrier. Pictures keep folks awake when reading specs._