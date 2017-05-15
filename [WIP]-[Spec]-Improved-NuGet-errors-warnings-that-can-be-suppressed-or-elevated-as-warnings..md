## Issue
[NuGet #4895](https://github.com/NuGet/Home/issues/4895): NET Core 2.0: Register all warnings/errors to assets file (including PackageTargetFallback)

## Problem
Warnings should be treated as errors or ignored based on developers' use-case. 
The following user scenarios are driving this feature: 
1. Package downgrades are warnings by default, can be overridden as errors, by the developer.
2. Package downgrades should be errors for .Net Core 2.0 projects. (As per requirements by the team) 
3. PackageTargetFallback warnings should be ignored for **certain** package dependencies if developer knows that a package is being brought into the project due to PackageTargetFallback and does not want be reminded again and again for each restore.
4. NuGet warnings should follow warning-levels defined in Project.Properties.Build just like any other warnings in the project.

## Scenarios
**Scenario-1** Package downgrades are warnings by default, can be overridden as errors, by the developer.
1. User has a ProjectA that has dependency on PackageZ version 7.1. 
2. ProjectA also has a dependency on ProjectB that has dependency on package PackageZ ver 7.8.
3. User builds ProjectA which in turn also builds PackageB and includes PackageZ
4. PackageZ version 7.1 is used and restored as part of ProjectA build due to the NuGet policy of – ‘Nearest dependency version wins’
5. User sees a warning being logged 
6. User can make this warning to be treated as error by going to Project.Properties.Build and specifying the warning number (NU1605) to be treated as error, henceforth.
![image](https://cloud.githubusercontent.com/assets/14800916/26081463/b1155498-397f-11e7-8c92-f832c1b71339.png)

**Scenario-2:** Package downgrades should be errors for .Net Core 2.0 projects.
1. User creates a new project that directly references “Microsoft.AspNetCore.Hosting” 1.0.0
2. User now adds a new reference “Microsoft.AspNetCore” 1.1.0 that references “Microsoft.AspNetCore.Hosting” 1.0.0
3. “Microsoft.AspNetCore.Hosting”  package currently gets pinned to the lower version, rather than getting lifted to the version depended upon by the Core package. 
4. User gets the downgrade error - This was originally a warning that this project type treats as an error.
5. User should be able to go to package reference properties, see this warning is an error and be able to clear it to make the current error back to warning.

**Scenario-3:** 3. PackageTargetFallback warnings should be ignored for **certain** package dependencies...
1. User creates a new NET Standard 2.0 project
2. User add NuGet reference to `Microsoft.Composition` package – this gets added due to PackageTargetFallBack
3. User sees the following warning(NU1701) as part of each build:
`Package 'Microsoft.Composition' was restored using 'net461' instead the project target framework 'netstandard2.0'. This may cause compatibility problems.`
4. User can suppress this warning(NU1701) totally for the project so that any warning w.r.t PackageTargetFallback is never shown for the project.
5. User can suppress this warning(NU1701) for only `Microsoft.Composition` package so that the warning does not show for `Microsoft.Composition` package but it will show for newer such packages added due to PackageTargetFallback. 
6. Suppression mechanisms (**TBD**)

## Who is the customer?
All developers who use NuGet in their projects.

## Evidence
As stated in the problem.

## Solution
* All the NuGet warnings will be numbered. List of all the numbered warnings can be found here - **TBD**
* These errors and warnings will be written into the assets file so that msbuild can output these errors appropriately.
* [**TBD**] UI experience for per package warning suppression in a project. 