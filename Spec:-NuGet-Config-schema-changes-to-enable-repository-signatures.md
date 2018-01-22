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

This spec tackles the first half of the solution i.e. update the schema for nuget.config file to be able to store repository trust information.
<br/>

### Repository Trust Information
We should store the following information to enable a trust relationship between a package consumer and a package repository.

* Repository key -  
Repository key will allow us to correlate a source to a trusted repository and help in cleanup in case for a source delete.

* Repository Source Certificate Endpoint URI - 
Repository Source Certificate Endpoint URI will allow us to communicate with the source to refresh certificate list. This is needed when a trusted repository is not a source and we have no other way of finding out the certificate endpoint.

* Repository Certificate Fingerprint -  
Repository Certificate Fingerprint will allow us to assert that the package was signed with a certificate that the repository is advertising in its certificate list. The certificate fingerprint should be calculated using a known algorithm as described below, in the `Repository Certificate Fingerprint Algorithm` point.

* Repository Certificate Fingerprint Algorithm - 
Repository Certificate Fingerprint Algorithm will allow us to use a known algorithm to calculate the certificate fingerprint. This should be set based on the value advertised by the package repository in it's certificate list. The set of accepted algorithms should contain - `SHA256`, `SHA384` and `SHA512`.
<br/>

### Repository Trust Information Location
Trust information for a repository should be stored along with the source information for package repositories i.e. nuget.config file. Ideally the trusted repository information should be defined in the same file as the corresponding source information is described.
<br/>

### Repository Trust Information Schema
Trust information for a repository should be stored in a similar fashion as [package source credentials](https://docs.microsoft.com/en-us/nuget/schema/nuget-config-file#packagesourcecredentials) - 

```
<packageSourceCredentials>
    <NuGet.org>
        <add key="Username" value="user@NuGet.org" />
        <add key="Password" value="..." />
    </NuGet.org>
</packageSourceCredentials>

```


```
<packageSourceTrustInformation>
    <NuGet.org>
        <add key="CertificatesBaseUrl" value="https://api.nuget.org/v3/repository-signature/CertificatesBaseUrl" />
        <add key="SHA256-Hash" value="jQCosvMgBxcgAQGNesKaHU1xvgly73B6jkRXZsf9Y8w=" />
        <add key="SHA384-Hash" value="vPv9/fx05OEc4atG7ny+5KXeLbV8xuZhp8ct1fgIhpfdP97ZQ2B801YBaBP61zd=" />
    </NuGet.org>
</packageSourceTrustInformation>
```
<br/>

### Repository Trust Information Gesture
To enable the following user gestures we need to update the existing [`nuget sources`](https://docs.microsoft.com/en-us/nuget/tools/cli-ref-sources) command and add a `nuget keys` command.

<br/>

#### Add a source as a trusted repository -  
This scenario should be supported to allow first class experience for package sources that can be added as a trusted repository.

`nuget sources add -Name NuGet.Org -Source https://api.nuget.org/v3/index.json --trusted`

The above command will create the following entries - 

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <packageSources>
        <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
    </packageSources>
    <packageSourceTrustInformation>
        <NuGet.org>
            <add key="CertificateBaseUrl" value="https://api.nuget.org/v3/<repository-signature>/<certificate-base-url>" />
            <add key="SHA256-Hash" value="jQCosvMgBxcgAQGNesKaHU1xvgly73B6jkRXZsf9Y8w=" />
            <add key="SHA384-Hash" value="vPv9/fx05OEc4atG7ny+5KXeLbV8xuZhp8ct1fgIhpfdP97ZQ2B801YBaBP61zd=" />
        </NuGet.org>
    </packageSourceTrustInformation>
</configuration>
```

The above command should fail if the repository does not support package signing -

```
nuget sources add -Name InSecureSource -Source https://api.insecuresource.org/v3/index.json --trusted

Package Source with Name: NuGet.Org cannot be added as a trusted repository. Please remove the --trusted switch.
```  

Further, deleting a package source should also delete the trusted source information - 

`nuget sources remove -Name NuGet.Org`

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources />
</configuration>
```
<br/>

#### Make a source a trusted repository - 
This scenario should be supported to allow users to make a package source as a trusted repository.

`nuget sources update -Name NuGet.Org --trusted`

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
    <packageSourceTrustInformation>
        <NuGet.org>
            <add key="CertificateBaseUrl" value="https://api.nuget.org/v3/<repository-signature>/<certificate-base-url>" />
            <add key="SHA256-Hash" value="jQCosvMgBxcgAQGNesKaHU1xvgly73B6jkRXZsf9Y8w=" />
            <add key="SHA384-Hash" value="vPv9/fx05OEc4atG7ny+5KXeLbV8xuZhp8ct1fgIhpfdP97ZQ2B801YBaBP61zd=" />
        </NuGet.org>
    </packageSourceTrustInformation>
</configuration>
```
<br/>

#### Add a trusted repository - 
This scenario should be supported to allow users to add a trusted repository which may not be a package source.

`nuget keys add -Name NuGet.org -Source https://api.nuget.org/v3/index.json`

This command will add an entry for trusted repository without adding a package source entry - 

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <packageSources />
    <packageSourceTrustInformation>
        <NuGet.org>
            <add key="CertificateBaseUrl" value="https://api.nuget.org/v3/<repository-signature>/<certificate-base-url>" />
            <add key="SHA256-Hash" value="jQCosvMgBxcgAQGNesKaHU1xvgly73B6jkRXZsf9Y8w=" />
            <add key="SHA384-Hash" value="vPv9/fx05OEc4atG7ny+5KXeLbV8xuZhp8ct1fgIhpfdP97ZQ2B801YBaBP61zd=" />
        </NuGet.org>
    </packageSourceTrustInformation>
</configuration>
```
<br/>

#### Sync keys for a trusted repository - 
This scenario should be supported to allow users to refresh the keys for a trusted repository which may not be a package source.

`nuget keys update -Name NuGet.org`

This command will update the entry for trusted repository without adding a package source entry - 

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <packageSources />
    <packageSourceTrustInformation>
        <NuGet.org>
            <add key="CertificateBaseUrl" value="https://api.nuget.org/v3/<repository-signature>/<certificate-base-url>" />
            <add key="SHA512-Hash" value="qt0apXB+hLPb7p1uawYX9ldV1F2aFR3tjGL41ojSL9w62IRm6Td9Eutg/l/cnSdCaUu88zRGNcS3Yw5odBCQTA==" />
        </NuGet.org>
    </packageSourceTrustInformation>
</configuration>
```
<br/>



