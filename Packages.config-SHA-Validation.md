# Packages.config SHA checking

* Status: **Reviewing**
* Author(s): [Andy Zivkovic](https://github.com/zivkan)

## Introduction

PackageReference projects recently got a [repeatable build feature](https://github.com/NuGet/Home/wiki/Enable-repeatable-package-restore-using-lock-file) which 1. lists the packages with all transient packages and 2. ensures that the restored package has the expected SHA hash.

Packages.config projects do not implicitly bring in transitive dependencies and does not support floating versions. At install time all dependencies are added to packages.config, including the version of each package, so the list of packages is already repeatable. However, there is value in also checking the SHA hashes to have confidence of build integrity across machines.

## Approach

In order to minimize differences between PackageReference and packages.config behavior, and to minimize disruption with future [centrally managed NuGet packages](https://github.com/NuGet/Home/wiki/Centrally-managing-NuGet-packages), the same `packages.lock.json` file will be used. This should minimize long-term engineering work and also provide the best customer experience. It will also simplify documentation as we won't need different documentation for PackageReference and packages.config projects, just small notes where the behavior is different.

### Feature Opt-In

Opting-in to the feature will be the same as repeatable buildings for PackageReference, by using either of the following options:

1. Add a `RestorePackagesWithLockFile` MSBuild property to the project with the value `true`.
2. The lock file already exists.

Packages.config lock file will use the same naming conventions as PackageReference lock files.

### Generating the lock file

The lock file will be created/updated in the following scenarios:

1. Any package is installed, updated or uninstalled in Visual Studio (either the Package Manager, or Package Manager Console).
2. Regenerate the lock file if it is out of sync with the project (different packages or package versions, including when lock file does not exist or is empty).
3. If it's easy to implement `nuget.exe restore -force-evaluate` will regenerate the lock file, to be consistent with PackageReference behavior, even if "force evaluate" doesn't entirely make sense in the context of packages.config.

### Locked mode

`nuget.exe` must accept the `-locked-mode` argument, and fail the restore instead of writing the lock file, which is the same behavior as `PackageReference`.

## Implementation details

### When and how the SHA is checked

When the feature is enabled, the content SHA of every package in a `packages.config` project will be checked on every restore, including `nuget.exe install`. To reduce performance impact, we will start writing `.nupkg.metadata` files in the solution packages folder, but only for packages where the content hash is required.

The performance impact of reading every package's `.nupkg.metadata` file on every restore will be evaluated and if necessary, a new file will be written to the intermediate output directory (`obj\` folder) to specify that restore has been completed. Presence of this file will prevent SHA validation on future restores. Since `packages.config` files that are not present with a .NET project (for example a `.csproj`) do not have a MSBuild project to get the intermediate output directory from, restoring these files will continue to validate on every restore.

### Recovering from failed restores due to content hash failures

The behavior will be the same as `PackageReference` projects.

## Example

Here is a sample packages.lock.json file

```json
{
  "version": 1,
  "dependencies": {
    ".NETFramework,Version=v4.7.2": {
      "Newtonsoft.Json": {
        "type": "Direct",
        "requested": "[12.0.1, 12.0.1]",
        "resolved": "12.0.1",
        "contentHash": "pBR3wCgYWZGiaZDYP+HHYnalVnPJlpP1q55qvVb+adrDHmFMDc1NAKio61xTwftK3Pw5h7TZJPJEEVMd6ty8rg=="
      }
    }
  }
}
```