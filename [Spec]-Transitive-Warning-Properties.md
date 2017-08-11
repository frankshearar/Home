This spec deals with the need for respecting the `NoWarn` properties from project references.

## Current Status

As per the work done in [restore warnings and errors](https://github.com/NuGet/Home/wiki/Restore-errors-and-warnings) users now experience NuGet warnings and errors as first class warnings and errors. Further, the work done in and [warning properties support](https://github.com/NuGet/Home/wiki/Improved-NuGet-warnings) we allow users to add the following 3 msbuild properties to control this experience -

* TreatWarningsAsErrors - This accepts true/false and can be used to treat all warnings as errors including NuGet/Msbuild/CSC.

  `<TreatWarningsAsErrors>True</TreatWarningsAsErrors>`

* WarningsAsErrors - This accepts a comma or semi-colon separated list of NuGet warning codes and can be used to convert specific warnings to errors.

  `<WarningsAsErrors>$(WarningsAsErrors);NU1603;NU1701</WarningsAsErrors>`

* NoWarn - This accepts a comma or semi-colon separated list of NuGet warning codes and can be used to hide specific warnings.

  `<NoWarn>$(NoWarn);NU1603;NU1605</NoWarn>`

* Package Specific NoWarn - This accepts a comma or semi-colon separated list of NuGet warning codes and can be used to hide specific warnings for a specific package reference. This property has to be set on a package reference.

  `<PackageReference Include="NuGet.Versioning" Version="4.1.5" NoWarn="NU1603" />`
  <br/>   or<br/>
  ```
  <PackageReference Include="NuGet.Versioning" Version="4.1.5">
    <NoWarn>NU1603</NoWarn>
  </PackageReference>
  ```


Combined Example - 
```
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>netcoreapp1.1;net46</TargetFramework>
    <NoWarn>$(NoWarn);NU1603;NU1605</NoWarn>
    <WarningsAsErrors>NU1603;NU1701</WarningsAsErrors>
    <TreatWarningsAsErrors>True</TreatWarningsAsErrors>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="NuGet.Versioning" Version="4.1.5" NoWarn="NU1603" />
  </ItemGroup>
</Project>
```

## Problem

Issue - https://github.com/NuGet/Home/issues/5501

Based on the work done above, we allow customers to control the warnings generated in one project. But currently we do not allow customers to control this experience for warnings being generated in a referenced project.

For example - `ProjA -> ProjB -> PkgX[NU1603]`
In the above scenario, we have Project A that references Project B which references Package X. Further, consider that Package X is a cause of a NuGet restore warning `NU1603`.

Based on the current status above we allow the users to turn off restore warnings in Project B either for all `NU1603` warnings by using the project wide `NoWarn` property. Or just for package X by using package specific `NoWarn` property.

But we do not allow users to control the same warnings in Project A. Thus when the user restores Project A, they need to turn off `NU1603` by using a project wide setting, which can hide other `NU1603` warnings generated in the project, Or add a direct reference to PackageX and then add a package specific `NoWarn`, which can alter the project's restore graph closure.


## Solution

The solution is to transitively pull in `NoWarn` properties from all the project references. The effect should be that if a child project has no-warn'd a warning then the parent project should not see the warning while restoring.

To achieve this we need to traverse the parent project's restore closure to pull in all the child project `NoWarn` and then apply them to the warnings as they are generated. 

## Implementation Details 

The current implementation is at https://github.com/NuGet/NuGet.Client/tree/dev-anmishr-p2pnowarn.

When a warning is generated in restore, it is passed to `RestoreCollectorLogger`. The logger then checks if it has the needed information to collect the transitive `NoWarn` properties i.e. If it has the parent project's `RestoreTargetGraph` and `PackageSpec`.

if the logger has the needed information, it invokes `TransitiveNoWarnUtils.CreatetransitiveWarningProperties`. In that method, for each restore target graphs we seed a queue with the parent project's direct dependencies. Then we traverse the closure in Breadth First style to visit each node in the closure. 
If the node is a project then we get the warning properties of the project and then merge them with the warning properties seen along the path taken to reach the node.
If the node is a package then we see if the path taken to this package has any `NoWarn` applicable to this package. If yes, then we intersect that with any `NoWarn` seen on all other paths to the package. If the result is an empty then we stop looking for that package because, we have 1 path with no `Nowarn` and thus it does not matter what other paths have since the warnings from the package will have at least one path for it's warnings.

The resulting collections are stored as a `Dictionary<string, HashSet<NuGetLogCode>>` resulting in a quick look up of a package ID to `NoWarn` Set of `NuGetLogCode`.

Once the resulting collection is generated, the same collection is used for all the other warnings generated as part of the restore.

## Open Questions

1. Turn off transitive NoWarn - 
Do we need a way for users to turn off the propagation of `NoWarn` transitively? If so the options are -
   * Add a switch to restore. - `restore --NotransitiveNoWarn`
   * An msbuild property that users can set to true if they do not want transitive `NoWarn`. - `msbuild /t:restores /p:NoTransitiveNoWarn=True`

_Discussed internally and decided that this is not needed._

2. Transitive `WarningsAsErrors` - 
Do we need to allow `WarningsAsErrors` to flow transitively as well?
e.g. - `A -> B[WarningsAsErrors=NU1603] -> PkgX[NU1603]`
In the above case, Project A references project B which has a reference to Package X. If Project B has set `NU1603` in `WarningsAsErrors` and package X brings in `NU1603`. Now if we restore Project A then it will trigger the restore for project B and project A. Project B restore will fail since project B has made the warning into an error. But Project A will succeed, since we do not look at transitive `WarningsAsErrors`.

_Discussed internally and decided that this is not needed as if one project restore fails then the solution restore will fail too and further, the parent project build will fail as well._

3. Apply package specific NoWarn to the package's closure - 
A user requested this scenario at https://github.com/NuGet/Home/issues/5740.

_We discussed this internally and currently we are not implementing this as this allows hiding warnings that slow down restore. However, we did discuss this as part of a larger work in near future._

4. Improve NuGet Error Experience by de-duping same warnings in errors - 
This came up during the discussion to improve user experience. I have filed a bug to track this - https://github.com/NuGet/Home/issues/5734

5. Double clicking on warnings and errors should take the user to the source - 
This came up during the discussion to improve user experience. I have filed a bug to track this - https://github.com/NuGet/Home/issues/5748