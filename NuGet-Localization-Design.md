## Problem
Design a localization strategy for NuGet package authors.

## Current Situation
Currently NuGet does not provide a strategy for shipping localized version of package for transitive project systems.
We do have official strategy for `Packages.Config` project [documented here](https://docs.microsoft.com/en-us/nuget/create-packages/creating-localized-packages).

## Proposals

As part of [sprint task](https://github.com/NuGet/Home/issues/5561) we discussed the following strategies for creating localized packages -
 
### Include all localized resources assemblies in a single package.

In this scenario, all the satellite assemblies can be part of the same package - 
```
lib
└───target_framework
    │   Code.dll
    │
    ├───culture_1
    │       Code.resources.dll
    │
    └───culture_2
            Code.resources.dll
```

### Create separate localized satellite package with all satellite assemblies.

In this scenario, all the satellite assemblies can be part of the new package which will deliver all the satellite assemblies as their pay load and **should** contain a package reference to the original package.

Original package structure - 
```
CodePackage.nupkg
└───lib
    └───target_framework
            Code.dll
```
Original package usage - 

```
...
  <ItemGroup>
    <PackageReference Include="CodePackage" Version="1.0.0"/>
  </ItemGroup>
...

```

Satellite package structure - 
```
CodePackage.Localized.nupkg
└───lib
    └───target_framework
        ├───culture_1
        │       Code.resources.dll
        │
        └───culture_2
                Code.resources.dll
```

Satellite package usage - 

```
...
  <ItemGroup>
    <PackageReference Include="CodePackage.Localized" Version="1.0.0"/>
  </ItemGroup>
...

```

### Create separate localized satellite packages, one for each localization culture.

In this scenario, all the satellite assemblies can be part of the separate packages, one for each culture. The satellite packages will deliver all the satellite assemblies as their pay load and **should** contain a package reference to the original package.

Original package structure - 
```
CodePackage.nupkg
└───lib
    └───target_framework
            Code.dll
```
Original package usage - 

```
...
  <ItemGroup>
    <PackageReference Include="CodePackage" Version="1.0.0"/>
  </ItemGroup>
...

```

Satellite package structure - 
```
CodePackage.Localized.culture_1.nupkg
└───lib
    └───target_framework
        └───culture_1
                Code.resources.dll
```

```
CodePackage.Localized.culture_2.nupkg
└───lib
    └───target_framework
        └───culture_2
                Code.resources.dll
```

Satellite package usage - 

```
...
  <ItemGroup>
    <PackageReference Include="CodePackage.Localized.culture_1" Version="1.0.0"/>
    <PackageReference Include="CodePackage.Localized.culture_2" Version="1.0.0"/>
  </ItemGroup>
...

```