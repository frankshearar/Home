Status: **Incubation**

## Issue
The work for this feature and the discussion around the spec is tracked here - **Enable repeatable builds for PackageReference based projects [#5602](https://github.com/NuGet/Home/issues/5602)**


# Context

Projects that use PackageReference to manage NuGet dependencies, only provide direct package dependencies. The transitive closure for the dependencies happen at the restore time.  
Refer to the dependency resolution algorithm for NuGet. Overall here is the summary of dependency resolution:

## Direct dependency resolution:
1. If exact version is specified - NuGet tries to resolve to the exact version. If not, it resolves to next highest version i.e. the lowest version equal to or near to the version specified. 
E.g. 
	
   `<PackageReference Include="My.Sample.Lib" Version="4.5.0" />`

   a. NuGet resolves to version 4.5.0 if present in the feed. 

   b. If Feed has only these versions: 4.0.0, 4.6.0, 5.0.0 then NuGet resolves to 4.6.0 

2. If a range or floating version is specified - NuGet resolves to the highest version specified in that range or that satisfies the floating expression.
E.g.

   a. Range is specified:
		
   `<PackageReference Include="My.Sample.Lib" Version="[4.0.0, 5.0.0]"/>`
		
      NuGet resolves to 5.0.0 is present and if not then resolves to the hishest version present in the specified range.
		
   b. Floating version is specified:
	
   `<PackageReference Include="My.Sample.Lib" Version="4.*"/>`
		
      NuGet resolves to the highest version that matches the floating expression. If not, then it resolves to next-highest-version-available* on the feed. 
	
      * This, IMO, is a bug: https://github.com/NuGet/Home/issues/5097
		
## Transitive dependency resolution
In case of transitive dependencies, the resolution is always to the lowest version specified in the dependency version or version ranges as specified here.

There are additional mechanisms to resolve conflict in dependency versions and those are resolved through "Nearest wins" and "Cousin dependencies" algorithm as discussed in details in the documentation.
		
# Problem
Users want their builds to be repeatable if the code doesn't change irrespective of when and where the build happens.
Refer to the context on NuGet dependency resolution. Because of the various ways by which dependency resolution happen, for a given project, the restore can bring in different versions of the package dependencies when run at different times and different places even when there is no change in package dependencies specified in the project file. This can be due to the following factors:
1. External: When package publishers change (add/remove) the packages on the feed(s).
2. Internal: When the nuget.config (if not checked-in) points to different feeds across builds.

This feature aspires to solve this issue and enable repeatable builds.

# Who is the customer?

While all users whose projects are PackageReference based would need this feature but for Enterprises this is really crucial (hygiene factor) for their CI/CD scenarios. 

# Evidence

We have had multiple internal partners reaching out to us from VS Team Services, Bing, Windows, Azure for this feature. Customers and community members have also asked for this feature. Refer to the following GitHub issues and comments on them:
* Lineups #2572 <https://github.com/NuGet/Home/issues/2572> 
* Why must resolve to Lowest Version? Allow users to determine package resolution strategy during package restore #5553 <https://github.com/NuGet/Home/issues/5553> 
* Twitter thread:
  * https://twitter.com/stimms/status/885268856960196612

There are a couple of related feature asks that are not part of this feature spec as mentioned below:
* Ability to lock package dependencies and their versions (version range) at solution/repo/global level
* Ability to define NuGet dependency version resolution when a version range is specified for direct and transitive dependencies.
	
# Solution

## High level proposal
Here is the idea (brain dump) till now:
1. Have a setting in nuget.config to enable repeatable build - TBD the mechanics
(Rest of the steps are assuming the repeatable build setting is allowed)
2. When restore happens, it creates a lock file in the root folder of the project. Here are the options:
   * Either the assets file can be cleaned up, modified and moved out from the obj directory to the root directory, OR
   * A new simplified file can be created in the root folder
3. Users should check-in this file to the repository
4. The lock file contains the following info:
   * All the direct dependencies with the versions (or ranges) as specified in the project file
   * All the direct dependencies with the actual version that was resolved during the restore.
   * All the transitive dependencies with the actual version resolved during the restore.
5. Subsequent restores will use this lock file to decide whether a new resolution needs to be done or not
   1. If the dependency versions have not changed with respect to what is specified in the lock file, use the lock file to get the package dependencies
   2. If there is a change between lock file and project file dependencies, it implies an explicit user action like update, add, delete (needs more thought) and hence re-generate the locks file.

## How does it help?
This ensures that any restore which can be part of different builds across place and time, will end up getting the same package dependencies.

Explicit user actions like update, add package dependency and remove package dependency will result in regeneration of the locks file. There would be a warning or info message displayed when this happens. 

It will also result in checkout of the locks file which means that the developer would know that the package dependencies may have changed. If this happens inadvertently, the lock file changes can be discarded and the original one restored to get the same repeatable build as before.

