* Status: **Reviewed**
* Author(s): [Ashish Jain](https://github.com/jainaashish)

## Issue

[2365](https://github.com/NuGet/Home/issues/2365) - Some assemblies in NuGet packages should be linked, not referenced

## Problem

Currently Project system has no say which assembly should be referenced vs linked when NuGet packages are consumed as `PackageReference`. This breaks important scenarios like when a Visual Studio SDK extension use a NuGet package such as `Microsoft.VisualStudio.Imaging.Interop.14.0.DesignTime` since the types from this package won't be embedded into the extension. Although the extension might work on that developer's machine (who has the VS SDK installed), the extension will fail at runtime on the end user's machine because that assembly isn't present. It was supposed to be embedded into the extension author's assembly.

## Who are the Customers

All NuGet users who uses PackageReference to manage their NuGet dependencies and wants to consume or produce Interop packages.

## Solution

NuGet will introduce a new folder structure called `/embed` very much similar to existing folder `/lib`. Package authors will keep all their embed interop types assemblies inside this new folder as well inside `/lib` folder (to support `packages.config` based projects) and non-embed assemblies will continue to go into existing folders. This way NuGet knows which assemblies to be referenced vs linked.

Next, NuGet will have a new section called `embed` in it's restore output file `obj/project.assets.json` for each embed type dependency apart from existing sections like `compile`, `runtime`, `build` etc.. This new section will let project system understand that assemblies inside it should be linked and not referenced in the consuming project. Since interop assemblies are duplicated in `/lib` folder as well so the same assemblies will also appear in `compile:` and `runtime:` section of `project.assets.json` file but those will be ignored.

### Sample nupkg structure for interop package

```
sample.interop.1.0.0.nupkg
 - sample.interop.nuspec
 - embed/
  - netstandard2.0/
   - sample.interop.dll
 - lib/
  - netstandard2.0/
   - sample.interop.dll
``` 

### Sample relevant portion of `project.assets.json` file

```
"targets": {
    ".NETStandard,Version=v2.0": {
    .................
      "sample.interop/1.0.0": {
        "type": "package",
        "embed": {
          "embed/netstandard2.0/sample.interop.dll": {}
        },
        "compile": {
          "lib/netstandard2.0/sample.interop.dll": {}
        },
        "runtime": {
          "lib/netstandard2.0/sample.interop.dll": {}
        }
      }
      ..............
    }
  }
```

## Change in `Pack` Command

Only change for `Pack` command will be to extend [existing MSBuild property](https://docs.microsoft.com/en-us/nuget/reference/msbuild-targets) `BuildOutputTargetFolder` to define one or more target folders to copy build output assemblies. Currently this property only allows single value but to author an interop package which works for both `packages.config` as well as `PackageReference` based projects, output assemblies must be copied in `/lib` (for packages.config) as well as `/embed` (for PackageReference). 

## No change in `IncludeTypes` Enum

There is an existing [include assets types](https://docs.microsoft.com/en-us/nuget/consume-packages/package-references-in-project-files) called `compile` which signifies which assmeblies from the package will be available for compilation and write code against. Embed types are no different and at no point in time, consumer needs to differentiate between what gets linked vs referenced in the consuming project. So these new embed assets will also be controled through `compile` flag. If a package consumer exclude `compile` assets then it will exlude any assets from `/lib` or `/embed` folders.

## No change for existing packages

There is no change for existing packages so the existing interop packages will continue to run with the current workaround to defining a target to set `<EmbedInteropTypes>true</EmbedInteropTypes>` for interop types assemblies. [NuGet sample](https://github.com/NuGet/Samples/tree/master/NuGet.Samples.Interop) which depict how to do this. So the new proposal will always be backward compatible and old NuGet clients will just ignore this new folder structure.

## Other solutions considered

There were few other proposals considered before finalizing this current solution of `/embed` folder. Here is the summary of those solutions:

#### 1. New metadata in .nuspec file 
This solution was to introduce a new metadata in nuspec file something like `Embed` which would be a boolean (similar to current `developmentDependency` metadata). So package author could set this to `true` to make a package as interop type. Unlike current solution, this proposal would not have required to duplicate assemblies in two folders (`/lib` and `/embed`), instead it would have continue to work with `/lib` folder. 

Only reason, we didn't choose this proposal, was that it would not have allowed to create a package with both interop assemblies as well as non-interop assemblies unlike current proposal where you can have interop assemblies in `/embed` folder and non-interop assemblies in `/lib` folder.

#### 2. Enhance `PackageTypes` metadata in .nuspec file
This solution was similar to the previous one, except instead of coming up with a new metadata, this proposal was to enhance existing `PackageTypes` metadata to introduce a new packageType (something like `InteropPackage`) which would have allowed the package authors to define a interop package. This proposal also had the same advantages/ disadvantages as the previous one.

#### 3. BuildAction for assembly files tag in .nuspec file
Another proposal was to define a buildAcion (similar to the buildAction for ContentFiles) for assembly files which would allow individual assembly file element to define a buildAction as `link`. This was a little complex solution of all the others besides the fact that there was no way to configure such elements through proejct file. All other proposals can easily work from project file without having an explicit .nuspec file, which is why we didn't pursue this one.
