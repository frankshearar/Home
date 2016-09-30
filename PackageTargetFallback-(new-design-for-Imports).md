## Problem
Sometimes, when attempting to use a package, the package doesn't have the appropriate TargetFramework supported. Imports provided a way for you to specify, for that case, which TargetFramework's assets to use instead - since you know they are compatible.

From: https://docs.nuget.org/ndocs/schema/project.json
> Imports
> Imports are designed to allow packages that use the dotnet TxM to operate with packages that don't declare a dotnet TxM. If your project is using the dotnet TxM then all the packages you depend on must also have a dotnet TxM, unless you add the following to your project.json in order to allow non dotnet platforms to be compatible with dotnet. If you are using the dotnet TxM then the PCL project system will add the appropriate imports statement based on the supported targets. 

> "frameworks": { 
>     "dotnet": { "imports" : "portable-net45+win81" } 
> } 

## Who is the customer?
Developers who are consuming NuGet packages.

## Solution
### MSBuild syntax to support PackageTargetFallback

    <PackageTargetFallback Condition="'$(TargetFramework)'=='Net45'">portable-net45+win81</PackageTargetFallback>

PackageTargetFallbacks may have been set in one of Microsoft targets (we are considering), or other ones. If you'd like to add the list already provided, you can add additional values to the PackageTargetFallback property:

    <PackageTargetFallback Condition="'$(TargetFramework)'=='Net45'">
        $(PackageTargetFallback);portable-net45+win8+wpa81+wp8
    </PackageTargetFallback >

## Replacing one library from a restore graph
If a restore is bringing the wrong assembly, it is possible to exclude that packages default choice, and replace it with your own choice. First with a top level packagereference, exclude all assets:
    <PackageReference Include="Newtonsoft.Json" Version="9.0.1">
        <ExcludeAssets>All</ExcludeAssets>
    </PackageReference>

Next, add your own reference to the appropriate local copy of the dll:
    <Reference Include="Newtonsoft.Json.dll" />

## Built in NuGet Knowledge about Compat & Fallbacks
* We need to confirm that NuGet code already knows all the guaranteed compatible TFMs - review with Eric St. John, and others.
* After we've confirmed our code is complete, we should consider adding some PackageTargetFallbacks into Microsoft targets as well.

## Other issues
* Will we create any UI in VS to expose the fact that PackageTargetFallbacks are used/specified: no
* We should ensure that NuGet logging output includes the fact that a PackageTargetFallback was used, since no compatible TFM was found.
