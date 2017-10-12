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

    -Verbosity <level> - Specifies the level of detail displayed in the output: quiet, normal, detailed

    -Signer "<cert_fingerprint>;..." - Verify that the signer certificate matches with one of the specified fingerprints.
                                     A certificate fingerprint is a SHA-1 hash of the certificate used to identify the certificate.
                                     If more than one fingerprint is provided it, the input should be a string separating each fingerprint with a semicolon.
```

### Return Value

Verify Command returns one of the following exit codes when it terminates.

| Exit code     | Description |
| ------------- | ------------- |
| 0  | Execution was successful.|
| 1  | Execution has failed. |

The errors and warnings will be displayed on the console.

### Verbosity details

The details that should be displayed on each verbosity level are described below. Each level should display the same as the level below plus whatever is specified in that level. In that sense, _quiet_ will be give the less amount of information, while _detailed_ the most.

**quiet**

* No output on successful execution and minimal output for failed execution.

**normal**

* Path to package being verified
* For each signature present
    * Type of signature (_author_ or _repository_)
    * Hash of signature
    * If the signature is valid/trusted.
    * Hashing algorithm used for signature
    * Signature Timestamp

**detailed**

* Information about each signing certificate and chain
    * Issued to
    * Issued by
    * Expires
    * Fingerprint
*  Information about each Timestamper certificate and chain
    * Issued to
    * Issued by
    * Expires
    * Fingerprint

Warnings are errors should be displayed if present no matter the verbosity level chosen.

### Errors and warnings

Some errors and warnings that should be displayed are:

**Warnings**

* A signing certificate doesn't chain up to a trusted root
* A timestamper certificate doesn't chain up to a trusted root

**Errors**

* Package not found
* Package is not signed
* The version of the signature is not supported
* The version of the manifest is not supported
* The hashing algorithm of the signature is not supported
* Signature file is too big
* Invalid signature. File might have been tampered.
* Signer certificate does not match any of the fingerprints provided.

### Corresponding commands

In future we would like to add support for the following platforms - 

* Dotnet CLI - `dotnet nuget verify <package_path> [Options]`

* MSBuild target - `msbuild /t:verifypackage <package_path> [Options]`

## Feedback
Please use the [tracking issue](https://github.com/NuGet/Home/issues/6005) to provide feedback or any questions that you might have. Thanks!
***