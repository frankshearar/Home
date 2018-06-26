Status: **Reviewing**

## Issue
Issue for spec - [NuGet/Home#6524](https://github.com/NuGet/Home/issues/6524)  
Parent spec - [Repository-Signatures](https://github.com/NuGet/Home/wiki/Repository-Signatures)  
Related spec - [NuGet-Package-Signing-Client-Policy](https://github.com/NuGet/Home/wiki/%5BSpec%5D-NuGet-Package-Signing-Client-Policy)

## Problem
As the next wave of package signing, consumers need to be able to trust specific package signers. Further, the trust information needs to be stored into the user's machine.

## Who is the customer?
All NuGet package consumers.

## Scenarios
* Enable package consumers to store repository and author trust information.

## Solution
* Update the schema for the nuget.config file to be able to store repository and author trust information.
* Define a gesture for users to be able to trust.

### Signers trust information

* Trusted signer key -  
Allows a user add a friendly name to identify the signer's trust information. If a trusted repository uses the same name as a source, it allows for correlation from a source to a trusted repository.

* Untrusted root -  
Indicates if it is allowed that a source's signing certificate chains to an untrusted root. Defaults to disallow.

Further for each certificate used by the repository, we should store the following - 

* Certificate fingerprint -  
Used to assert the package being verified has been signed by the trusted repository. The fingerprint should be a calculated based on the `Repository Certificate Fingerprint Algorithm` described below.


* Certificate fingerprint algorithm -  
Allows for crypto-agility while calculating the certificate fingerprint. The algorithm currently supported are - 
  * `SHA256`
  * `SHA384`
  * `SHA512`

### Repository specific trust information
We should store the following information to enable a trust relationship between a package consumer and a package repository.

* Repository source service index URI -  
Allows communication with the source to refresh certificate list and to match with the `V3ServiceIndex` attribute in a repository signature. This is needed when a trusted repository is not a source so there is no other way of finding out the certificate endpoint. If this property is present in a trusted signer entry, the entry is taken to be a repository.

If the user wants to only trust specific package owners for a repository, they should be able to specify a list of trusted owners that will be compared against the `Package Owners` field in the repository signature metadata.

### Trust information location
Trust information should be stored in the nuget.config file.

### Repository trust information schema
```xml
<trustedSigners>
  <NAME>
    <add key="type" value="TRUSTED_SIGNER_TYPE" />
    <add key="serviceIndex" value="SERVICE_INDEX_URI" /> <!-- If present then type should be Repository -->
    <add key="CERT_HASH" 
         value="FINGERPRINT_ALGORITHM"
         untrustedRoot="UNTRUSTED_ROOT" />
    <add key="owners" value="LIST_OF_TRUSTED_OWNERS" /> <!-- Can only be present if type is Repository -->
  </NAME>
</trustedSigners >
```

For example -

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <trustedSigners>
    <NuGet.Org>
      <add key="type" value="repository" />
      <add key="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
           value="SHA256" />
      <add key="vPv9/fx05OEc4atG7ny+5KXeLbV8xuZhp8ct1fgIhpfdP97ZQ2B801YBaBP61zd=" 
           value="SHA384" />
      <add key="owners" value="aspnet;microsoft" />
    </NuGet.Org>
    <vsts>
      <add key="type" value="repository" />
      <add key="serviceIndex"
           value="https://api.vsts.com/feed/index.json" />
      <add key="OdiswAGAy7da6Gs6sghKmg9e9r90wM385jRXZsf9Y5q="
           value="SHA256" />
      <add key="untrustedRoot" value="allow" />
    </vsts>
    <Microsoft>
      <add key="type" value="author" />
      <add key="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
           value="SHA256" />
    </Microsoft>
    <PatoBeltran>
      <add key="type" value="author" />
      <add key="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
           value="SHA256" />
      <add key="untrustedRoot" value="allow" />
    </PatoBeltran>
  </trustedSigners>
</configuration>
```

### Trust information gesture
To enable the following user gestures we need to create a new `nuget trusted-signers` command.
<br/>


#### Summary - 

| Operation | Signer Type | Command | Remarks |
| --- | --- | --- | --- |
| List | All | `nuget trusted-signers` |
| Add | Repository | `nuget trusted-signers Add -Name <n> [-Owners <o>]` | Only works if there exists a source with the same name |
| Add | Repository | `nuget trusted-signers Add -Name <n> -ServiceIndex <s> [-Owners <o>]` |
| Add | Author | `nuget trusted-signers Add -Name <n> -CertificateFingerprint <f> -FingerprintAlgorithm <a> [-UntrustedRoot <u>]` |
| Add | Repository | `nuget trusted-signers Add <package_path> -Repository -Name <n> [-Owners <o>] [-UntrustedRoot <u>]` | Only works if package is repository signed or repository countersigned |
| Add | Author | `nuget trusted-signers Add <package_path> -Author -Name <n> [-UntrustedRoot <u>]` | Only works if package is author signed |
| Remove | Any | `nuget trusted-signers Remove -Name <n>` |
| Sync | Repository | `nuget trusted-signers Sync -Name <n>` | Refreshes certificates entries with the ones announced by the repository.<br />The entry has to exist and be a trusted repository with a service index or a corresponding package source. |


This gesture will be translated to dotnet CLI by modifying the nuget add, nuget  `nuget trusted-signers` command.
<br/>


#### Summary - 

| Operation | Signer Type | Command | Remarks |
| --- | --- | --- | --- |
| List | All | `nuget trusted-signers` |
| Add | Repository | `dotnet nuget add trusted-signers -Name <n> [-Owners <o>]` | Only works if there exists a source with the same name |
| Add | Repository | `dotnet nuget add trusted-signers -Name <n> -ServiceIndex <s> [-Owners <o>]` |
| Add | Author | `dotnet nuget add trusted-signers -Name <n> -CertificateFingerprint <f> -FingerprintAlgorithm <a> [-UntrustedRoot <u>]` |
| Add | Repository | `dotnet nuget add trusted-signers <package_path> -Repository -Name <n> [-Owners <o>] [-UntrustedRoot <u>]` | Only works if package is repository signed or repository countersigned |
| Add | Author | `dotnet nuget add trusted-signers <package_path> -Author -Name <n> [-UntrustedRoot <u>]` | Only works if package is author signed |
| Remove | Any | `dotnet nuget remove trusted-signers -Name <n>` |
| Sync | Repository | `dotnet nuget sync trusted-signers -Name <n>` | Refreshes certificates entries with the ones announced by the repository.<br />The entry has to exist and be a trusted repository with a service index or a corresponding package source. |

### `sync` action



### Open questions

- Given the current inheritance model of the nuget.config, what happens when two configs at different levels have a `trustedSigner` element with the same name?
- What happens when there exist multiple entries with the same `serviceIndex`, different name value, but with conflicting certificate elements? (e.g. same `certificateFingerprint` but different `untrustedRoot` value)
- Is the schema proposed the most readable/user friendly? Is there a way to make it less verbose and still have a deterministic experience for the user.
- If the sync action automatically refreshes the certificates list in a trusted repository entry with the ones announced by the server, should there be a gesture to let the update the `untrustedRoot` setting on each certificate given by the server?
- Given that `type` value is based on the presence of `serviceIndex`, should `serviceIndex` be an additional property of the type element?
- If user has a different config on two folders inside a solution, given that we don’t have granularity of which package asked for a specific package to be downloaded, what trusted signers will be used when verifying each of the packages downloaded?

