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
* Define a gesture for users to be able to trust signers.

### Signers trust information

* Trusted signer key -  
Allows a user add a friendly name to identify the signer's trust information.

* Trusted signer type - 
Identifies the trusted signer with a type. The only permitted values are `author` and `repository`. If the value is `repository` a `serviceIndex` element should exist.

Further for each certificate used by the repository, we should store the following - 

* Certificate fingerprint -  
Used to assert the package being verified has been signed by the trusted repository. The fingerprint should be a calculated based on the `Repository Certificate Fingerprint Algorithm` described below.

* Certificate fingerprint algorithm -  
Allows for crypto-agility while calculating the certificate fingerprint. The algorithm currently supported are - 
  * `SHA256`
  * `SHA384`
  * `SHA512`

* Untrusted root -  
Indicates if it is allowed or disallowed that this certificate chains to an untrusted root.

### Repository specific trust information
We should store the following information to enable a trust relationship between a package consumer and a package repository.

* Repository source service index URI -  
Allows communication with the source to refresh certificate list and to match with the `V3ServiceIndex` attribute in a repository signature. If this property is present in a trusted signer entry, the entry is taken to be a repository.

If the user wants to only trust specific package owners for a repository, they should be able to specify a list of trusted owners that will be compared against the `Package Owners` field in the repository signature metadata.

### Trust information location
Trust information should be stored in the nuget.config file.

### Trust information schema

The current design and implementation of the nuget.config has some limitations, there a two approaches that can be followed in the implementation of the trust information schema:

1. Following the current implementation

```xml
<trustedSigners>
  <NAME>
    <add key="type" 
         value="TRUSTED_SIGNER_TYPE"
         serviceIndex="SERVICE_INDEX_URI" /> <!-- If present then value should be Repository -->
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
      <add key="type" 
           value="repository"
           serviceIndex="https://api.nuget.org/v3/index.json" />
      <add key="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
           value="SHA256"
           untrustedRoot="disallow" />
      <add key="vPv9/fx05OEc4atG7ny+5KXeLbV8xuZhp8ct1fgIhpfdP97ZQ2B801YBaBP61zd=" 
           value="SHA384"
           untrustedRoot="disallow" />
      <add key="owners" value="aspnet;microsoft" />
    </NuGet.Org>
    <vsts>
      <add key="type"
           value="repository"
           serviceIndex="https://api.vsts.com/feed/index.json" />
      <add key="OdiswAGAy7da6Gs6sghKmg9e9r90wM385jRXZsf9Y5q="
           value="SHA256"
           untrustedRoot="allow" />
    </vsts>
    <Microsoft>
      <add key="type" value="author" />
      <add key="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
           value="SHA256"
           untrustedRoot="disallow" />
    </Microsoft>
    <PatoBeltran>
      <add key="type" value="author" />
      <add key="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
           value="SHA256"
           untrustedRoot="allow" />
    </PatoBeltran>
  </trustedSigners>
</configuration>
```

2. Refactoring nuget.config

This approach would require to refactor the current implementation of nuget.config to have more freedom around the schema inside the config file. The advantage of this approach includes better readability, deterministic clear behavior in each section and more freedom on which attribute to use as the key. Also, by using this approach we enable nuget.config to be updated in the future to follow a better and more powerfull schema.

```xml
<trustedSigners>
  <repository name="NAME" serviceIndex="SERVICE_INDEX_URI">
    <certificate fingerprint="CERT_HASH" 
                 hashAlgorithm="FINGERPRINT_ALGORITHM"
                 allowUntrustedRoot="UNTRUSTED_ROOT_BOOL" /><!-- defaults to false -->
    <owners>OWNER_1;OWNER_2;...;OWNER_N</owners>
  </repository>
  <author name="NAME">
    <certificate fingerprint="CERT_HASH" 
                 hashAlgorithm="FINGERPRINT_ALGORITHM"
                 allowUntrustedRoot="UNTRUSTED_ROOT_BOOL" /><!-- defaults to false -->
  </author>
</trustedSigners>
```

**Notes on schema:**
- `trustedSigners` section should have the ability to be cleared.
- `clear` should not work inside a trusted signer entry.
- `repository` entries should be unique based on the `serviceIndex`.
- `author` entries should be unique based on the `name`.
- If two trusted signer entries are found to have the same unique key on different levels of the hierarchy, the furthest from the user should be overridden.
- If there are multiple certificates with the same fingerprint (e.g. multiple different trusted signer entries can share the same certificate) and conflicting `allowUntrustedRoot` values, a warning should be generated and the most restrictive setting should be used.

For example -

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <trustedSigners>
    <repository name="NuGet.Org" serviceIndex="https://api.nuget.org/v3/index.json">
      <certificate fingerprint="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
                   hashAlgorithm="SHA256"
                   allowUntrustedRoot="false" />
      <certificate fingerprint ="vPv9/fx05OEc4atG7ny+5KXeLbV8xuZhp8ct1fgIhpfdP97ZQ2B801YBaBP61zd=" 
                   hashAlgorithm="SHA384"
                   allowUntrustedRoot="false" />
      <owners>aspnet;microsoft</owners>
    </repository>
    <repository name="vsts" serviceIndex="https://api.vsts.com/feed/index.json">
      <certificate fingerprint="OdiswAGAy7da6Gs6sghKmg9e9r90wM385jRXZsf9Y5q="
                   hashAlgorithm="SHA256"
                   allowUntrustedRoot="true" />
    </repository>
    <author name="Microsoft">
      <certificate fingerprint="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
                   hashAlgorithm="SHA256"
                   allowUntrustedRoot="false" />
    </author>
    <author name="PatoBeltran">
      <certificate fingerprint="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
                   hashAlgorithm="SHA256"
                   allowUntrustedRoot="true" />
    </author >
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
| Add | Author | `nuget trusted-signers Add -Name <n> -CertificateFingerprint <f> -FingerprintAlgorithm <a> [-UntrustedRoot <u>]` | If entry with the same name exists, append the new certificate element.</br > `untrustedRoot` defaults to `disallow`
| Add | Repository | `nuget trusted-signers Add <package_path> -Repository -Name <n> [-Owners <o>] [-UntrustedRoot <u>]` | Only works if package is repository signed or repository countersigned.<br /> `untrustedRoot` defaults to `disallow` |
| Add | Author | `nuget trusted-signers Add <package_path> -Author -Name <n> [-UntrustedRoot <u>]` | Only works if package is author signed.<br /> `untrustedRoot` defaults to `disallow` |
| Remove | Any | `nuget trusted-signers Remove -Name <n>` |
| Sync | Repository | `nuget trusted-signers Sync -Name <n>` | Refreshes certificates entries with the ones announced by the repository.<br /> The entry has to exist and be a trusted repository with a service index or a corresponding package source. |


This gesture will be translated to dotnet CLI by updating the `dotnet nuget add`, `dotnet nuget remove` commands and add a `dotnet nuget sync` and a `dotnet nuget list` commands.
<br/>

#### Summary - 

| Operation | Signer Type | Command | Remarks |
| --- | --- | --- | --- |
| List | All | `dotnet nuget list trusted-signers` |
| Add | Repository | `dotnet nuget add trusted-signers -Name <n> [-Owners <o>]` | Only works if there exists a source with the same name |
| Add | Repository | `dotnet nuget add trusted-signers -Name <n> -ServiceIndex <s> [-Owners <o>]` |
| Add | Author | `dotnet nuget add trusted-signers -Name <n> -CertificateFingerprint <f> -FingerprintAlgorithm <a> [-UntrustedRoot <u>]` | If entry with the same name exists, append the new certificate element.</br > `untrustedRoot` defaults to `disallow`
| Add | Repository | `dotnet nuget add trusted-signers <package_path> -Repository -Name <n> [-Owners <o>] [-UntrustedRoot <u>]` | Only works if package is repository signed or repository countersigned.<br /> `untrustedRoot` defaults to `disallow` |
| Add | Author | `dotnet nuget add trusted-signers <package_path> -Author -Name <n> [-UntrustedRoot <u>]` | Only works if package is author signed.<br /> `untrustedRoot` defaults to `disallow` |
| Remove | Any | `dotnet nuget remove trusted-signers -Name <n>` |
| Sync | Repository | `dotnet nuget sync trusted-signers -Name <n>` | Refreshes certificates entries with the ones announced by the repository.<br /> The entry has to exist and be a trusted repository with a service index or a corresponding package source. |

### `sync` action

Over the course of time, a repository could deprecate or add certificates to their list of supported certificates. The `sync` action is designed as a way for the user to explicitly update their trust to that specific repository. This action will send a request to the appropriate service index to get a list of certificates that will replace the current trusted certificates for the corresponding trusted repository entry.

### Open questions

- Given the current inheritance model of the nuget.config, what happens when two configs at different levels have a `trustedSigner` element with the same name?
<br />**PB:** Each `trustedSigner` element should be atomic, therefore if an entry that has the same name is found in the hierarchy it should override the previous one found.

- What happens when there exist multiple entries with the same `serviceIndex`, different name value, but with conflicting certificate elements? (e.g. same `certificateFingerprint` but different `untrustedRoot` value)
<br />**PB:** We should consider `serviceIndex` to be the unique key for a repository trusted signer entry, therefore it should only exist one entry per `serviceIndex`. However, if two certificates are found with conflicting `untrustedRoot` we should warn and take the most restrictive one.

- Given that `type` value is based on the presence of `serviceIndex`, should `serviceIndex` be an additional property of the type element?
<br />**PB:** The `type` element will always be author unless there exists a `serviceIndex`, therefore I think it would be a good idea to have them both in the same element, this way we eliminate the possibility of `serviceIndex` being in a different config than `type`.

- Is the schema proposed the most readable/ user-friendly? Is there a way to make it less verbose and still have a deterministic experience for the user.
- If the sync action automatically refreshes the certificates list in a trusted repository entry with the ones announced by the server, should there be a gesture to let the update the `untrustedRoot` setting on each certificate given by the server?
- If a user has a different config on two folders inside a solution, given that we don’t have the granularity of which package asked for a specific package to be downloaded, what trusted signers will be used when verifying each of the packages downloaded?
- The current design only lets the user to add a trusted author with a single certificate and hand edit if more than one certificate should be trusted. Is there a way to create a "batch add" to let the user add a trusted author with multiple certificates?
<br />**PB:** If a user does a `nuget trusted-signer add` with a certificate information an a name of an existing entry, it should append that certificate to the existing entry.

- Owners in a repository signature are not limited to a set of characters, therefore it is possible that a package owner in a signature has the character we chose as a delimiter (i.e. semicolon - `;`). Should we make the schema take a single line for each owner?
<br />**PB:** Based on the assumption that owners will usually be repository usernames, they will be constrained by the repository to a certain set of characters, therefore choosing a semicolon to delimit this list should be safe.