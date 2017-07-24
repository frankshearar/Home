## Problem
Design a localization strategy for NuGet package authors.

## Current Situation
Currently NuGet does not provide a strategy for shipping localized version of package for transitive project systems.
We do have official strategy for `Packages.Config` project [documented here](https://docs.microsoft.com/en-us/nuget/create-packages/creating-localized-packages).

## Proposals

As part of [sprint task](https://github.com/NuGet/Home/issues/5561) we discussed the following strategies for creating localized packages - 
1. Include all localized resources assemblies in a single package.

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

2. Create separate localized satellite package with all satellite assemblies.

In this scenario, all the satellite assemblies can be part of the new package which will deliver all the satellite assemblies as the pay load and will contain a package reference to the original package- 
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

3. Create separate localized satellite packages, one for each localization culture.

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