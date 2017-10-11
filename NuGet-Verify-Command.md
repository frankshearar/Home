**Status**: Review

## Issue
[Task for Specing](https://github.com/nuget/home/issues/6005) and [Task for execution](https://github.com/nuget/home/issues/6006)

## Problem
Signed packages help with authenticity and integrity of a package when it is being consumed by NuGet users. Currently there is no way to verify that a signature in a package is valid. 

## Who is the customer?
All NuGet package authors.

## Evidence
Part of the larger [package signing](https://github.com/NuGet/Home/wiki/Author-Package-Signing) effort.

##  Solution
We will add a first level command to NuGet.exe which will allow package authors and package sources to verify signed NuGet packages.

### Command Signature 
```
usage: NuGet verify <package_path>  [options]

Verifies a NuGet package.

argument:

    package_path -  Path to the package(s) that needs to be verified.

options:

    -Verbosity <level> - Specifies the level of detail displayed in the output: normal, quiet, detailed

    -Signer <cert_hash> … - Verify that the signer certificate matches with one of the specified hashes. 
```

### Return Value

Verify Command returns one of the following exit codes when it terminates.

| Exit code     | Description |
| ------------- | ------------- |
| 0  | Execution was successful.|
| 1  | Execution has failed. |

The errors and warnings will be displayed on the console.

### Validation on Verify

* If the package passed to the command is unsigned it should fail.

### Corresponding commands

In future we would like to add support for the following platforms - 

* Dotnet CLI - `dotnet nuget verify <package_path> [Options]`

* MSBuild target - `msbuild /t:verifypackage <package_path> [Options]`

## Feedback
Please use the [tracking issue](https://github.com/NuGet/Home/issues/6005) to provide feedback or any questions that you might have. Thanks!
***