Status: **Incubation**

## Issue
Issue for spec - [NuGet/Home#6419](https://github.com/NuGet/Home/issues/6419)  
Parent spec - [Repository-Signatures](https://github.com/NuGet/Home/wiki/Repository-Signatures)

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

* Repository Certificate Fingerprint -  
Repository Certificate Fingerprint will allow us to assert that the package was signed with a certificate that the repository is advertising in its certificate list. The fingerprint should be a SHA256 fingerprint.

### Repository Trust Information Location
Trust information for a repository should be stored along with the source information for package repositories i.e. nuget.config file.

### Repository Trust Information Schema
```
  <trustedRepositories>
    <repository key="NuGet.org" [serviceIndex="https://api.nuget.org/v3/index.json"]>
      <certificate name="CN=NuGet.Org" fingerprint="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w="/>
      <certificate name="CN=NuGet.Org Other Certificate" fingerprint="vPv9/fx05OEc4atG7ny+5KcgQGNesKaHU1AxvXB6W2d="/>
    </repository>
    <repository key="VSTS" [serviceIndex="https://vsts.com/feed/index.json"]>
      <certificate name="CN=VSTS" fingerprint="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w="/>
    </repository>
  <trustedRepositories>
```

### Repository Trust Information Gesture
To enable the following user gestures we need to update the existing [`nuget sources`](https://docs.microsoft.com/en-us/nuget/tools/cli-ref-sources) command. All of the following operations are performed on `%AppData%\NuGet\NuGet.config` by default. Users can control the config file using the `-configFile` parameter.
<br/>

#### Add a source [No change in behavior]-  
`nuget sources add -Name NuGet.Org -Source https://api.nuget.org/v3/index.json` 

The above command will create the following entries - 

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
</configuration>
```
<br/>

#### Add a source as a trusted repository -  
`nuget sources add -Name NuGet.Org -Source https://api.nuget.org/v3/index.json -WithTrust` 

The above command will create the following entries - 

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <trustedRepositories>
    <repository key="NuGet.org" [serviceIndex="https://api.nuget.org/v3/index.json"]>
      <certificate name="CN=NuGet.Org" fingerprint="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w="/>
    </repository>
  <trustedRepositories>
</configuration>
```

The above command should fail if the repository does not support package signing -

```
nuget sources add -Name InSecureSource -Source https://api.insecuresource.org/v3/index.json -trusted

Package Source with Name: InSecureSource cannot be added as a trusted repository as it does not support repository signing. Please remove the '-trusted' option.
```  
<br/>

#### Deleting a package source - 
`nuget sources remove -Name NuGet.Org`

Before -
```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <trustedRepositories>
    <repository key="NuGet.org" [serviceIndex="https://api.nuget.org/v3/index.json"]>
    <certificate name="CN=NuGet.Org" fingerprint="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w="/>
    </repository>
  <trustedRepositories>
</configuration>
```

After - 
```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources />
  <trustedRepositories>
    <repository key="NuGet.org" [serviceIndex="https://api.nuget.org/v3/index.json"]>
      <certificate name="CN=NuGet.Org" fingerprint="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w="/>
    </repository>
  <trustedRepositories>
</configuration>
```

<br/>

#### Deleting a package source and trust information- 
`nuget sources remove -Name NuGet.Org -WithTrust`

Before -
```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <trustedRepositories>
    <repository key="NuGet.org" [serviceIndex="https://api.nuget.org/v3/index.json"]>
      <certificate name="CN=NuGet.Org" fingerprint="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w="/>
    </repository>
  <trustedRepositories>
</configuration>
```

After - 
```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources />
</configuration>
```

<br/>

#### Make a source a trusted repository - 
`nuget sources update -Name NuGet.Org -WithTrust`

Before -
```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
</configuration>
```

After - 
```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <trustedRepositories>
    <repository key="NuGet.org" [serviceIndex="https://api.nuget.org/v3/index.json"]>
      <certificate name="CN=NuGet.Org" fingerprint="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w="/>
    </repository>
  <trustedRepositories>
</configuration>
```
<br/>

#### Add a trusted repository - 
`nuget sources add -Name NuGet.org -Source https://api.nuget.org/v3/index.json -OnlyTrust`

This command will add an entry for trusted repository without adding a package source entry - 

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources />
  <trustedRepositories>
    <repository key="NuGet.org" [serviceIndex="https://api.nuget.org/v3/index.json"]>
      <certificate name="CN=NuGet.Org" fingerprint="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w="/>
    </repository>
  <trustedRepositories>
</configuration>
```
<br/>

#### Refresh certificates for a trusted repository - 
`nuget sources update -Name NuGet.Org -WithTrust`

This command will update the entry for trusted repository - 

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources />
  <trustedRepositories>
    <repository key="NuGet.org" [serviceIndex="https://api.nuget.org/v3/index.json"]>
      <certificate name="CN=NuGet.Org" fingerprint="jQCosvMgBxcgQGNesKaHU1Axvgly73B6jkRXZsf9Y8w="/>
    </repository>
  <trustedRepositories>
</configuration>
```
<br/>

#### Trusted repositories in Visual Studio - 
We should add support for the following in Visual Studio NuGet options control - 

* Display if a package source is trusted repository.

* Allow users to make package source into a trusted repository.
<br/>

## Open Questions