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
Based on the work done above, we allow customers to control the warnings generated in one project. But currently we do not allow customers to control this experience for warnings being generated in a referenced project.

For example - `ProjA -> ProjB -> PkgX[NU1603]`
In the above scenario, we have Project A that references Project B which references Package X. Further, consider that Package X is a cause of a NuGet restore warning NU1603.

Based on the current status above we allow the users to turn off restore warnings in Project B either for all NU1603 warnings by using the project wide `NoWarn` property. Or just for package X by using package specific `NoWarn` property.

But we do not allow users to control the same warnings in Project A. Thus when the user restores Project A, they need to turn off `NU1603` by using a project wide setting, which can hide other `NU1603` warnings generated in the project, Or add a direct reference to PackageX and then add a package specific `NoWarn`, which can alter the project's restore graph closure.

## Solution
