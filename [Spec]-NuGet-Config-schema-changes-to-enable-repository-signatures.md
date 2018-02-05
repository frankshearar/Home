Status: **Incubation**

## Issue
Issue for spec - [NuGet/Home#6419](https://github.com/NuGet/Home/issues/6419)  
Parent spec - [Repository-Signatures](https://github.com/NuGet/Home/wiki/Repository-Signatures)  
Related Spec - [NuGet-Client-Security-Policy](https://github.com/NuGet/Home/wiki/%5BSpec%5D-NuGet-Client-Security-Policy)

## Problem
Once we enable repository package signing, we need to enable consumers to be able to trust a package repository. Further, the trust information needs to be stored into the users machine.

## Who is the customer?
All NuGet package consumers.

## Scenarios
* Enable package consumers to store repository trust information

## Solution
* Update the schema for nuget.config file to be able to store repository trust information.
* Define a gesture for users to be able to trust a package repository.
<br/>

### Repository Trust Information
We should store the following information to enable a trust relationship between a package consumer and a package repository.

* Repository key -  
Repository key will allow us to correlate a source to a trusted repository and help in cleanup in case for a source delete.

* Repository Source Service Index URI - 
Repository Source Service Index URI will allow us to communicate with the source to refresh certificate list. This is needed when a trusted repository is not a source and we have no other way of finding out the certificate endpoint.

Further for each certificate used by the repositoy, we should store the folling - 

* Repository Certificate Fingerprint -  
Repository Certificate Fingerprint will allow us to assert that the package was signed with a certificate that the repository is advertising in its certificate list. The fingerprint should be a calculated based on the `Repository Certificate Fingerprint Algorithm` described below.

* Repository Certificate SubjectName -  
Repository Certificate SubjectName will allow users to recognize certificates more easily as subject names are human readable.

* Repository Certificate Fingerprint Algorithm -  
Repository Certificate Fingerprint Algorithm will allow us to calculate the hash algorithm used while claculating the certificate fingerprint. The algorithm should be one of the following - 
  * `SHA256`
  * `SHA384`
  * `SHA512`

### Repository Trust Information Location
Trust information for a repository should be stored along with the source information for package repositories i.e. nuget.config file.

### Repository Trust Information Schema
```xml
  <trustedSources>
    <NuGet.Org>
      <add key="serviceIndex" value="SERVICE_INDEX_URI" />
      <add key="CERT_HASH" 
           value="SUBJECT_NAME" 
           fingerprintAlgorithm="FINGERPRINT_ALGORITHM" />
    </NuGet.Org>
  </trustedSources>
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
      <add key="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w=" 
           value="CN=NuGet.Org" 
           fingerprintAlgorithm="SHA256" />
      <add key="vPv9/fx05OEc4atG7ny+5KXeLbV8xuZhp8ct1fgIhpfdP97ZQ2B801YBaBP61zd=" 
           value="CN=NuGet.Org NewCert" 
           fingerprintAlgorithm="SHA384" />
    </NuGet.Org>
    <vsts>
      <add key="serviceIndex" 
           value="https://api.vsts.com/feed/index.json" />
      <add key="OdiswAGAy7da6Gs6sghKmg9e9r90wM385jRXZsf9Y5q=" 
           value="CN=vsts" 
           fingerprintAlgorithm="SHA256" />
    </vsts>    
  </trustedSources>
</configuration>
```

### Repository Trust Information Gesture
To enable the following user gestures we need to update the existing [`nuget sources`](https://docs.microsoft.com/en-us/nuget/tools/cli-ref-sources) command. All of the following operations are performed on `%AppData%\NuGet\NuGet.config` by default. Users can control the config file using the `-configFile` parameter.
<br/>

#### Summary - 

| Operation         	| Source Type       	| Command                                               | Remarks             	|
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
nuget sources add -Name InSecureSource -Source http://source.org/v3/index.json -Trust

Package Source with Name: InSecureSource cannot be added as a trusted repository.
InSecureSource does not support repository signing. 
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
_Ankit: It is user friendly as users don't need to add a `-Trust` switch. But it can be a security concern that we should not add trust without user confirmation/action_  

* Should we delete repository trust information on source delete?  
_Ankit: No, since we are not adding the trust information implicitly, we should not delete it implicitly._  

* How should the client trust NuGet.org?  
_Rido: Client should always trust NuGet.org unless the the user explicitly untrusts it._
```xml
<untrustedSources>
  <add key="NuGet.Org" value="true" />    
</untrustedSources>
```
* Do we need the service index in the trusted source?  
_Ankit: It should be an optional entry to be added only if there is no corresponding source. That will allow us to refresh the certificate list without asking the user for a url._  
_Rido: Not needed. The failed restore will display the URI for which it should be certificates need to be refreshed._