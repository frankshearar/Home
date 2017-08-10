# Current Status
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

# Problem
Based on the work done above, we allow customers to control the warnings generated in one project. But currently we do not allow customers 