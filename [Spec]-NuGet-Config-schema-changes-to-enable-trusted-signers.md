Status: **Incubation**

## Issue
Issue for spec - [NuGet/Home#6419](https://github.com/NuGet/Home/issues/6419)  
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

### Author and repository trust information

* Trusted signer key -  
Allows a user add a friendly name to identify the signer's trust information. If a trusted repository uses the same name as a source, it allows for correlation from a source to a trusted repository.

* Allow untrusted root -  
Indicates if it is allowed that a source's signing certificate chains to an untrusted root. Defaults to false.

Further for each certificate used by the repository, we should store the following - 

* Certificate fingerprint -  
Used to assert the package being verified has been signed by the trusted repository. The fingerprint should be a calculated based on the `Repository Certificate Fingerprint Algorithm` described below.

* Certificate SubjectName -  
Allows users to recognize certificates more easily as subject names are human readable.

* Certificate fingerprint algorithm -  
Allows for crypto-agility while calculating the certificate fingerprint. The algorithm currently supported are - 
  * `SHA256`
  * `SHA384`
  * `SHA512`

### Repository specific trust information
We should store the following information to enable a trust relationship between a package consumer and a package repository.

* Repository source service index URI -  
Allows communication with the source to refresh certificate list and to match with the `V3ServiceIndex` attribute in a repository signature. This is needed when a trusted repository is not a source so there is no other way of finding out the certificate endpoint.

If the user wants to only trust specific package owners for a repository, they should be able to specify a list of trusted owners that will be compared against the `Package Owners` field in the repository signature metadata.

### Trust information location
Trust information should be stored in the nuget.config file.

### Repository trust information schema
```xml
<trustedSources>
  <SOURCE_NAME>
    <add key="requireTrustedRoot" value="REQUIRE_TRUST_BOOL" />
    <add key="serviceIndex" value="SERVICE_INDEX_URI" />
    <add key="owners" value="LIST_OF_TRUSTED_OWNERS" />
    <add key="CERT_HASH" 
         value="SUBJECT_NAME" 
         fingerprintAlgorithm="FINGERPRINT_ALGORITHM" />
  </SOURCE_NAME>
</trustedSources>
<trustedAuthors>
  <AUTHOR_NAME>
    <add key="requireTrustedRoot" value="REQUIRE_TRUST_BOOL" />
    <add key="CERT_HASH"
         value="SUBJECT_NAME"
         fingerprintAlgorithm="FINGERPRINT_ALGORITHM" />
  </AUTHOR_NAME>
</trustedAuthors>
```

For example -
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <trustedSources>
    <NuGet.Org>
      <add key="owners" value="aspnet;microsoft" />
      <add key="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
           value="CN=NuGet.Org" 
           fingerprintAlgorithm="SHA256" />
      <add key="vPv9/fx05OEc4atG7ny+5KXeLbV8xuZhp8ct1fgIhpfdP97ZQ2B801YBaBP61zd=" 
           value="CN=NuGet.Org NewCert"
           fingerprintAlgorithm="SHA384" />
    </NuGet.Org>
    <vsts>
      <add key="requireTrustedRoot"
           value="false" />
      <add key="serviceIndex"
           value="https://api.vsts.com/feed/index.json" />
      <add key="OdiswAGAy7da6Gs6sghKmg9e9r90wM385jRXZsf9Y5q="
           value="CN=vsts"
           fingerprintAlgorithm="SHA256" />
    </vsts>    
  </trustedSources>
  <trustedAuthors>
    <Microsoft>
      <add key="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
           value="CN=Microsoft" 
           fingerprintAlgorithm="SHA256" />
    </Microsoft>
    <PatoBeltran>
      <add key="requireTrustedRoot"
           value="false" />
      <add key="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
           value="CN=Pato's self-signed cert" 
           fingerprintAlgorithm="SHA256" />
    </PatoBeltran>
  </trustedAuthors>
</configuration>
```

### Trust information gesture
To enable the following user gestures we need to create a new `nuget trust` command.
<br/>


#### Summary - 

| Operation         	| Signer Type       | Command                                               | Remarks
|-------------------	| --------------    |------------------------------------------------------ | ---------------
| Add               	| Repository        | `nuget trust add -Repository -Name <n> [-Owners <o>] [-AllowUntrustedRoot <u>]` | Only works if there exists a source with the same name
| Add               	| Repository        | `nuget trust add -Repository -Name <n> -ServiceIndex <s> [-Owners <o>] [-AllowUntrustedRoot <u>]`
| Add               	| Repository        | `nuget trust add -Repository -Name <n> -CertificateFingerprint <f> -FingerprintAlgorithm <a> -CertificateSubjectName <sn> [-ServiceIndex <s>] [-Owners <o>] [-AllowUntrustedRoot <u>]`
| Add                   | Repository        | `nuget trust add -Repository -Name <n> <Package_path> [-Owners <o>] [-AllowUntrustedRoot <u>]`   | Only works if package is repository signed or repository countersigned
| Add                   | Author            | `nuget trust add -Author -Name <n> <Package_path> [-AllowUntrustedRoot <u>]` | Only works if package is author signed
| Add                   | Author            | `nuget trust add -Author -Name <n> -CertificateFingerprint <f> -FingerprintAlgorithm <a> -CertificateSubjectName <sn> [-AllowUntrustedRoot <u>]` |
| Remove            	| Any               | `nuget trust remove -Name <n>` |  The entry has to exist
| Remove            	| Any               | `nuget trust remove -Name <n> -CertificateFingerprint <f>` | Removes a specific certificate entry from an existing trusted signer entry
| Update trust entry               	| Any        | `nuget trust update -Name <n> -CertificateFingerprint <f> -FingerprintAlgorithm <a> -CertificateSubjectName <sn>` | Adds a certificate entry to an existing trusted signer         	             	
| Update trust entry    | Repository        | `nuget trust update -Name <n>`             	         | Refreshes certificates entries with the ones announced by the repository. The entry has to exist and be a trusted repository with a service index or a corresponding package source.
| Update trust entry    | Repository        | `nuget trust update -Name <n> -ServiceIndex <s> [-Owners <o>] [-AllowUntrustedRoot <u>]` | The entry has to exist and be a trusted repository
| Update trust entry    | Any        | `nuget trust update -Name <n> -AllowUntrustedRoot <u>` | The entry has to exist
<br/>

----
WIP

#### Trusted repositories in Visual Studio - 
We should add support for the following in Visual Studio NuGet options control - 

* Display the config file(s) for each package source.

* Display if a package source is trusted repository e.g. - 

![](https://github.com/NuGet/Home/wiki/Repo-Signature-media/VSConfigUI.png)

* Allow users to make package source into a trusted repository.

* Display Trusted repositories trusted repositories.
<br/>

## Open Questions

* Should we add a package source as a trusted repository by default?  
_Ankit: It is user-friendly as users don't need to add a `-Trust` switch. But it can be a security concern that we should not add trust without user confirmation/action_  

* How should the client trust NuGet.org?  
_Rido: Client should always trust NuGet.org unless the user explicitly untrusts it._
```xml
<untrustedSources>
  <add key="NuGet.Org" value="true" />    
</untrustedSources>
```
* Once nuet.org starts signing, what does the client do?
Possibilities: Update the config file. Keep the keys in memory. Track untrusted as well.

* Do we need the service index in the trusted source?  
_Ankit: It should be an optional entry to be added only if there is no corresponding source. That will allow us to refresh the certificate list without asking the user for a URL._  
_Rido: Not needed. The failed restore will display the URI for which it should be certificates need to be refreshed._

* Should the user be able to add and remove trusted repositories from the sources command at the same time they add or remove a source?

#### Add a source as a trusted repository -  
`nuget sources add -Name NuGet.Org -Source https://api.nuget.org/v3/index.json -Trust` 

The above command will create the following entries - 

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <trustedSources>
    <NuGet.Org>
      <add key="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
            value="CN=NuGet.Org" 
            fingerprintAlgorithm="SHA256" />
    </NuGet.Org>
  </trustedSources>
</configuration>
```

The above command should fail if the repository does not support package signing -

```
nuget sources add -Name InsecureSource -Source http://source.test/v3/index.json -Trust

Package Source with Name: InsecureSource cannot be added as a trusted repository.
InsecureSource does not support repository signing. 
Please remove the '-Trust' option.
```  

```
nuget sources add -Name FileShare -Source \\share -Trust

Package Source with Name: FileShare cannot be added as a trusted repository.
FileShare does not support repository signing. 
Please remove the '-Trust' option.
```  
<br/>

#### Removing a package source and trust information- 
`nuget sources remove -Name NuGet.Org -Trust`

Before -
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <trustedSources>
    <NuGet.Org>
      <add key="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
           value="CN=NuGet.Org" 
           fingerprintAlgorithm="SHA256" />
    </NuGet.Org>
  </trustedSources>
</configuration>
```

After - 
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources />
</configuration>
```

<br/>