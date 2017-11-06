## Issue
The work for this feature and the discussion around the spec is tracked here - **TBD**

All related specs/issues at a glance: 

| Title | Issue | Spec |
|:------------- |:-------------:| -----:|
| Enable repeatable builds for PackageReference based projects (via lock file)  | [#5602](https://github.com/nuget/home/issues/5602) | [Incubation](https://github.com/NuGet/Home/wiki/Enable-repeatable-builds-for-PackageReference-based-projects) |
| **Manage allowed packages for a solution (or globally)**  | TBD |  **[Incubation](https://github.com/NuGet/Home/wiki/Manage-allowed-packages-for-a-solution-%28or-globally%29)** |
| Allow users to determine package resolution strategy during package restore - direct or transitive | [#5553](https://github.com/nuget/home/issues/5553) | TBD |

## Problem
For a large repo with multiple solutions or a big solution with multiple projects, it becomes challenging to manage package dependencies and especially managing consistent packages' versions across the projects in the solution/repo. 
Many a times organizations/teams know about a subset of package versions that are good to use and would like the same package versions to be used throughout the projects in their solutions/repos. 
Today it is non trivial to manage these scenarios with existent NuGet tooling.

## Who is the customer?
Users with large repo consisting of multiple solutions or a big solution with multiple projects who want to enforce a set of allowed packages/versions usage for all the projects/solutions/repos

## Evidence


## Solution

The proposal is to provide NuGet tooling that lets users manage **allowed** packages for their projects. The proposal is optimized for a VS solution but works for larger code-bases that has multiple repos/solutions.

Following are the requirements:
1. Users should be able to specify a set of packages that can be used as direct dependency for a project.
2. Users should be able to specify a set of packages that can be used as a dependency - direct/transitive for a project.
3. Users should be able to specify a version or version range for packages to be used for projects in a solution/repo.
4. [**Not-MVP**] Users should be able to specify the package source for each of the packages he/she lists as allowed packages, as describe above


