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

* Require trusted root -  
Indicates if a source should require to have its signing certificate to chain to a trusted root. Defaults to true.

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
To enable the following user gestures we need to update the existing [`nuget sources`](https://docs.microsoft.com/en-us/nuget/tools/cli-ref-sources) command.
<br/>

#### Summary - 

| Operation         	| Source type       	| Command                                               | Remarks             	|
|-------------------	|-------------------	|-------------------------------------------------------|---------------------	|
| Add               	| Source            	| `nuget sources add -Name <n> -Value <v>`        	    | Do not add trust    	|
| Add               	| Source with trust 	| `nuget sources add -Name <n> -Value <v> -Trust` 	    |                     	|
| Remove            	| Source            	| `nuget sources remove -Name <n>`                    	| Do not remove trust 	|
| Remove            	| Source with trust 	| `nuget sources remove -Name <n> -Trust`             	|                     	|
| Update to Trusted 	| Source            	| `nuget sources update -Name <n> -Trust`             	|                     	|
| Refresh Trust     	| Source            	| `nuget sources update -Name <n> -Trust`             	|                     	|
<br/>

#### Add a source [No change in behavior]-  
`nuget sources add -Name NuGet.Org -Source https://api.nuget.org/v3/index.json` 

The above command will create the following entries - 

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
</configuration>
```
<br/>

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

#### Removing a package source [No change in behavior] - 
`nuget sources remove -Name NuGet.Org`

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
  <trustedSources>
    <NuGet.Org>
      <add key="serviceIndex" 
           value="https://api.nuget.org/v3/index.json" />    
      <add key="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
           value="CN=NuGet.Org" 
           fingerprintAlgorithm="SHA256" />
    </NuGet.Org>
  </trustedSources>
</configuration>
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

#### Make a source a trusted repository - 
`nuget sources update -Name NuGet.Org -Trust`

Before -
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
</configuration>
```

After - 
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
<br/>

#### Add a source as trusted repository only - 
`nuget sources add -Name NuGet.org -Source https://api.nuget.org/v3/index.json -Trust`  
`nuget sources remove -Name NuGet.org`


The above commands will add an entry for trusted repository without adding a package source entry - 
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources />
  <trustedSources>
    <NuGet.Org>
      <add key="serviceIndex" 
           value="https://api.nuget.org/v3/index.json" />    
      <add key="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
           value="CN=NuGet.Org" 
           fingerprintAlgorithm="SHA256" />
    </NuGet.Org>
  </trustedSources>
</configuration>
```
<br/>

#### Refresh certificates for a trusted repository - 
`nuget sources update -Name NuGet.Org -Trust`

This command will update the entry for trusted repository - 

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <trustedSources>
    <NuGet.Org>
      <add key="vPv9/fx05OEc4atG7ny+5KXeLbV8xuZhp8ct1fgIhpfdP97ZQ2B801YBaBP61zd=" 
           value="CN=NuGet.Org NewCert" 
           fingerprintAlgorithm="SHA384" />
    </NuGet.Org>
  </trustedSources>
</configuration>
```
<br/>

#### Trusted repositories in Visual Studio - 
We should add support for the following in Visual Studio NuGet options control - 

* Display the config file(s) for each package source.

* Display if a package source is trusted repository e.g. - 

![](https://github.com/NuGet/Home/wiki/Repo-Signature-media/VSConfigUI.png)

* Allow users to make package source into a trusted repository.

* Display Trusted repositories trusted repositories.
<br/>

## Open Questions

* Should we encourage hand edits for the nuget.config file?  
_Ankit: No, we explicitly discourage that in our [docs](https://docs.microsoft.com/en-us/nuget/consume-packages/configuring-nuget-behavior#changing-config-settings) because starting v3.4.3, malformed config files are silently ignored._

* Should we add a package source as a trusted repository by default?  
_Ankit: It is user-friendly as users don't need to add a `-Trust` switch. But it can be a security concern that we should not add trust without user confirmation/action_  

* Should we delete repository trust information on source delete?  
_Ankit: No, since we are not adding the trust information implicitly, we should not delete it implicitly._  

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