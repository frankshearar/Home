# NuGet Package Sources Diagnostics

## Problem
Customers often complain of performance issues with retrieving packages in search, installing and updating packages. Often when we analyze these individual issues, we find that in a large percentage of cases the issues users are seeing are due to the following. Package Sources here refer to:- File shares, NuGet.Server's, 3rd party repositories and NuGet.org.

  * NuGet package sources response times are slow
  * Nuget package sources are behind a corporate firewall
  * User is offline
  * The backing file shares of NuGet.org are either not optimized or very large.

We want to proactively inform the users of these issues, so they can take corrective action and/or give them an option to temporarily disable certain sources for a period of time of the session.

## Who is the customer?
Any customer who has a slow network, works offline and online, has NuGet.Server installations and have package sources behind corporate firewalls.

## Evidence
TBD

## Solution

### Scenarios that hit a file share/ server (ordered in terms of priority)
 
  * Update: Checking for updates. Fetching and installing packages.
  * Package restore
  * Browse/Search; Search in package manager UI and list in PMC
  * Install
  * Detailed metadata fetch (Click on package)
  * PowerShell autocomplete
  * Consolidate

### Diagnostics Scenarios (Ordered in terms of priority)

  * Server is not accessible: Server is not available or server can't be reached
  * Server is slow: Package has a large number of versions (1000+ CI feeds). Server implementation is not optimal for large version set
  * FileShare is not efficient
  * vNext (Requires proxy rules): Semantic understanding of what packages are being downloaded per source

### Priority of experiences
  * VS
  * PMC
  * NuGet.xplat
  * Nuget.exe

### Walkthrough
  * User opens Visual Studio
  * User open  package manager UI
  * Sees an update and hits install
  * We detected 1-N issues
  * We bring up a UI that shows what the issues are
  * This UI will have single disable for duration command, disable for all time command
  * On clicking this package manager enters into temp disable mode, we show some indicator that you are this in this mode.
  * You can either get out of the temp disable mode by using some command in VS or restart VS
  * If the user clicks permanently disable, then there is no highlight forever and goes through normal flow.