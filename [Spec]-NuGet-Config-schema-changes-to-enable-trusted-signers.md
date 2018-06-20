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
    <add key="type" value="TRUSTED_SIGNER_TYPE" /> <!-- Defaults to Author -->
    <add key="untrustedRoot" value="UNTRUSTED_ROOT" /> <!-- Defaults to disallow -->
    <add key="serviceIndex" value="SERVICE_INDEX_URI" /> <!-- If present then type is Repository -->
    <add key="owners" value="LIST_OF_TRUSTED_OWNERS" /> <!-- Can only be present if type is Repository -->
    <add key="CERT_HASH" 
         value="FINGERPRINT_ALGORITHM" />
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
      <add key="owners" value="aspnet;microsoft" />
      <add key="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
           value="SHA256" />
      <add key="vPv9/fx05OEc4atG7ny+5KXeLbV8xuZhp8ct1fgIhpfdP97ZQ2B801YBaBP61zd=" 
           value="SHA384" />
    </NuGet.Org>
    <vsts>
      <add key="type" value="repository" />
      <add key="serviceIndex"
           value="https://api.vsts.com/feed/index.json" />
      <add key="untrustedRoot" value="allow" />
      <add key="OdiswAGAy7da6Gs6sghKmg9e9r90wM385jRXZsf9Y5q="
           value="SHA256" />
    </vsts>
    <Microsoft>
      <add key="type" value="author" />
      <add key="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
           value="SHA256" />
    </Microsoft>
    <PatoBeltran>
      <add key="untrustedRoot" value="allow" />
      <add key="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
           value="SHA256" />
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
| Add | Repository | `nuget trusted-signers Add -Name <n> [-Owners <o>] [-UntrustedRoot <u>]` | Only works if there exists a source with the same name |
| Add | Repository | `nuget trusted-signers Add -Name <n> -ServiceIndex <s> [-Owners <o>] [-UntrustedRoot <u>]` |
| Add | Author | `nuget trusted-signers Add -Name <n> -CertificateFingerprint <f> -FingerprintAlgorithm <a> [-UntrustedRoot <u>]` |
| Add | Repository | `nuget trusted-signers Add <package_path> -Repository -Name <n> [-Owners <o>] [-UntrustedRoot <u>]` | Only works if package is repository signed or repository countersigned |
| Add | Author | `nuget trusted-signers Add <package_path> -Author -Name <n> [-UntrustedRoot <u>]` | Only works if package is author signed |
| Remove | Any | `nuget trusted-signers Remove -Name <n>` |
| Remove | Author | `nuget trusted-signers Remove -Name <n> -CertificateFingerprint <f>` | Removes a specific certificate entry from an existing trusted signer entry |
| Update | Author | `nuget trusted-signers Update -Name <n> -CertificateFingerprint <f> -FingerprintAlgorithm <a>`| Adds a certificate entry to an existing trusted signer |
| Update | Repository | `nuget trusted-signers Update -Name <n>` | Refreshes certificates entries with the ones announced by the repository.<br />The entry has to exist and be a trusted repository with a service index or a corresponding package source. |
| Update | Repository | `nuget trusted-signers Update -Name <n> -Owners <o> [-UntrustedRoot <u>]` | The entry has to exist and be a trusted repository |
| Update | Any | `nuget trusted-signers Update -Name <n> -UntrustedRoot <u>` | The entry has to exist |