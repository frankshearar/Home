# NuGet Package Debugging Improvements in NuGet Client

## Issue

[6104](https://github.com/NuGet/Home/issues/6104) - Improve NuGet Package Debugging and symbols experience

## Related spec

[Spec](https://github.com/NuGet/Home/wiki/NuGet-Package-Debugging-&-Symbols-Improvements)

## Problem

Currently NuGet has no well-defined experience for publishing a symbols package, what it should contain and how it should be consumed. The Package Manager UI (and NuGet.org)do not indicate whether symbols are available for a particular package. The default symbol server for .symbols.nupkg packages is a 3rd party service and not owned/maintained by NuGet.

## Who are the customers?

The customers are both consumers and authors of packages who want first class debugging experience from referenced packages.

## Solution

This spec details improvements and changes to the NuGet Client (NuGet.exe) commands push and pack to better support creating symbols package as listed in the related [spec](https://github.com/NuGet/Home/wiki/NuGet-Package-Debugging-&-Symbols-Improvements) .

### Package Authoring Experience

NuGet.exe pack and msbuild /t:pack will have an additional command line property called ```-SymbolsPackageFormat``` . Since the symbols package will not be produced by default (as per current behavior), this option will only be applicable if passed along with ```-Symbols``` in NuGet.exe or ```--include-symbols``` in dotnet.exe. SymbolsPackageFormat property can have one of the two values: snupkg, symbols.nupkg. The default value of this property would be snupkg.

#### SymbolsPackageFormat : snupkg

When snupkg is passed (explicitly or because of defaults), the result will be two files: a .nupkg, and a .snupkg . The .nupkg file would be exactly the same as it is today, but the .snupkg file would have the following characteristics:

1) The .snupkg will have the same id and version as the corresponding .nupkg.
2) The .snupkg will have the exact folder structure as the nupkg for any DLL or EXE files with the distinction that instead of DLLs/EXEs, their corresponding PDBs will be included in the same folder hierarchy. Files and folders with extensions other than PDB will be left out of the snupkg.
3) The .nuspec file in the .snupkg will also specify a new PackageType as follows:
``` 
<packageTypes>
  <packageType name="SybmolsPackage"/>
</packageTypes>
```
4) If an author decides to use a custom nuspec to build their nupkg and snupkg, the snupkg should have the same folder hierarchy and files detailed in 2).
5) ```authors``` and ```owners``` field will be excluded from the snupkg's nuspec.

#### SymbolsPackageFormat : symbols.nupkg

This option is only being provided for compat reasons with the current behavior. Using this in conjunction with -Symbols or --include-symbols will ensure the symbols.nupkg is created in the same way as it is today.

### Package Publishing Experience

In order to create a seamless experience to publish symbols package (aka snupkgs), the snupkg file will be pushed to NuGet.org by default if it exists right next to the corresponding nupkg file. Package authors will also have the option to separately upload the snupkg file using the push command. The NuGet.org endpoint will automatically figure out the match between nupkg and snupkg from the Id and version of both packages. If you use API keys, you will use the same API key for both the NuGet package and the Symbols package. 

If the nupkg and snupkg are being pushed in the same command, the nupkg will always be pushed and validated before NuGet attempts to push a snupkg package.
