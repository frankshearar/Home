## Issue
[NuGet #4895](https://github.com/NuGet/Home/issues/4895): NET Core 2.0: Register all warnings/errors to assets file (including PackageTargetFallback)

## Problem
The following user scenarios are driving this feature: 
1. A NuGet warning can be overridden as errors, by the developer from Project properties and/or csproj file.
2. A NuGet warning can be suppressed by the developer from Project properties and/or csproj file. 
3. Per Package NuGet warning can be suppressed by the developer from Project properties and/or csproj file.
4. NuGet warnings should follow warning-levels defined in Project just like any other warnings in the project.

**Scenario-1:** A NuGet warning can be overridden as errors, by the developer from Project properties and/or csproj file
1. User creates a new project that references `NuGet.Packaging 3.5.0` that references `NuGet.Versioning 3.5.0`
2. User now another package dependency `NuGet.Commands 4.0.0` that has dependency to `NuGet.Versioning 4.0.0` as shown in the dependency tree:
`NuGet.Commands 4.0.0 -> NuGet.Configuration 4.0.0 -> NuGet.Versioning 4.0.0`
3. User gets a downgrade warning (NU1605):

```
Detected package downgrade: NuGet.Versioning from 4.0.0 to 3.5.0
  NuGet.Packaging 3.5.0 -> NuGet.Versioning 3.5.0
  NuGet.Commands 4.0.0 -> NuGet.Configuration 4.0.0 -> NuGet.Versioning 4.0.0
```

4. User now adds the warning (NU1605) as error in the Project.Properties.Build UI:
![image](https://cloud.githubusercontent.com/assets/14800916/26081463/b1155498-397f-11e7-8c92-f832c1b71339.png)
5. Alternatively, user can write the following in the csproj file to treat this warning as error:
 `<TreatSpecificWarningsAsErrors>NU1605</TreatSpecificWarningsAsErrors>`
6. User now sees this downgrade as an error.

**Scenario-2:** A NuGet warning can be suppressed by the developer from Project properties and/or csproj file. 
1. User creates a new NET Standard 2.0 project
2. User add NuGet reference to `Microsoft.Composition` package – this gets added due to PackageTargetFallBack
3. User sees the following warning(NU1701) as part of each build:
`Package 'Microsoft.Composition' was restored using 'net461' instead the project target framework 'netstandard2.0'. This may cause compatibility problems.`
4. User can suppress this warning(NU1701) totally for the project so that any warning w.r.t PackageTargetFallback is never shown for the project in Project.Properties.Build UI as follows:<br>
![image](https://cloud.githubusercontent.com/assets/14800916/26125901/7623489a-3a38-11e7-8604-d90be0fb6a49.png)
5. Alternatively, user can suppress this warning(NU1701) by using the following line in the csproj file:
 `<NoWarn>NU1701</NoWarn>`

**Scenario-3:** Per Package NuGet warning can be suppressed by the developer from Project properties and/or csproj file.
1. User creates a new NET Standard 2.0 project
2. User add NuGet reference to `Microsoft.Composition` package – this gets added due to PackageTargetFallBack
3. User sees the following warning(NU1701) as part of each build:
`Package 'Microsoft.Composition' was restored using 'net461' instead the project target framework 'netstandard2.0'. This may cause compatibility problems.`
4. User can suppress this warning(NU1701) specific to the package 'Microsoft.Composition' in the package dependency property:

![image](https://cloud.githubusercontent.com/assets/14800916/26465230/568f8aa8-413f-11e7-91b1-0378b987ddc9.png)
5. User can do the same by specifying the suppression in the csproj file (comma or semi-colon separated list of warnings:
```
    <PackageReference Include="Contoso.Base.API" Version="1.0.3">
      <NoWarn>NU1701</NoWarn>
    </PackageReference>
```

**Scenario-4:** NuGet warnings should follow warning-levels defined in Project just like any other warnings in the project.
1. User creates a new NET Standard 2.0 project
2. User goes to Project.Properties.Build and sets the Warning level as 1:
![image](https://cloud.githubusercontent.com/assets/14800916/26126231/b07a654a-3a39-11e7-8ea9-d7c13c004e3d.png)
3. Alternatively, in the csproj file, the users sets the following to set the Warning level as 1:
  `<WarningLevel>1</WarningLevel>`
4. User now only sees severe warnings and not the other warnings.
**TBD** Classification of warnings into levels 1 through 4.

## Who is the customer?
All developers who use NuGet using PackageReference. .NET Core, .NET Standard and other projects (opted-in to use PackageReference).

## Evidence
As stated in the problem.

## Solution
* All the NuGet warnings will be numbered. List of all the numbered warnings can be found here - [Restore errors and warnings](https://github.com/NuGet/Home/wiki/Restore-errors-and-warnings)
* These errors and warnings will be written into the assets file so that msbuild can output these errors appropriately.

## Appendix
## NET Core 2.0 scenarios
1. Package downgrades should be errors for .Net Core 2.0 projects. (As per requirements by the team) 
2. PackageTargetFallback warnings should be ignored for **certain** package dependencies if developer knows that a package is being brought into the project due to PackageTargetFallback and does not want be reminded again and again for each restore.

**Scenario-1:** Package downgrades should be errors for .Net Core 2.0 projects.
1. User creates a new project that directly references “Microsoft.AspNetCore.Hosting” 1.0.0
2. User now adds a new reference “Microsoft.AspNetCore” 1.1.0 that references “Microsoft.AspNetCore.Hosting” 1.0.0
3. “Microsoft.AspNetCore.Hosting”  package currently gets pinned to the lower version, rather than getting lifted to the version depended upon by the Core package. 
4. User gets the downgrade error - This was originally a warning that this project type treats as an error.
5. User should be able to go to package reference properties, see this warning is an error and be able to clear it to make the current error back to warning.

**Scenario-2:** 3. PackageTargetFallback warnings should be ignored for **certain** package dependencies...
1. User creates a new NET Standard 2.0 project
2. User add NuGet reference to `Microsoft.Composition` package – this gets added due to PackageTargetFallBack
3. User sees the following warning(NU1701) as part of each build:
`Package 'Microsoft.Composition' was restored using 'net461' instead the project target framework 'netstandard2.0'. This may cause compatibility problems.`
4. User can suppress this warning(NU1701) totally for the project so that any warning w.r.t PackageTargetFallback is never shown for the project.
5. User can suppress this warning(NU1701) for only `Microsoft.Composition` package so that the warning does not show for `Microsoft.Composition` package but it will show for newer such packages added due to PackageTargetFallback. 
6. Suppression mechanisms (**TBD**)