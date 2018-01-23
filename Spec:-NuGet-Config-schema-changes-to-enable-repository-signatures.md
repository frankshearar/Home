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

This spec tackles the first part of the solution in detail i.e. update the schema for nuget.config file to be able to store repository trust information. This spec also, briefly, defines a solution for the second part of the solution i.e. define a gesture for users to be able to trust a package repository.
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

### Repository Trust Information Location
Trust information for a repository should be stored along with the source information for package repositories i.e. nuget.config file. Ideally the trusted repository information should be defined in the same file as the corresponding source information is described.

### Repository Trust Information Schema
Trust information for a repository should be stored in a similar pattern as [package source credentials](https://docs.microsoft.com/en-us/nuget/schema/nuget-config-file#packagesourcecredentials) - 

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

> We should also consider writing the OID for the hash algorithm, in the same way as we do while creating a package signature content. Though that makes the file, less human friendly.

### Repository Trust Information Gesture
To enable the following user gestures we need to update the existing [`nuget sources`](https://docs.microsoft.com/en-us/nuget/tools/cli-ref-sources) command and add a `nuget keys` command.

<br/>

#### Add a source as a trusted repository -  
This scenario should be supported to allow first class experience for package sources that can be added as a trusted repository.

`nuget sources add -Name NuGet.Org -Source https://api.nuget.org/v3/index.json`

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

#### Refresh keys for a trusted repository - 
This scenario should be supported to allow users to refresh the keys for a trusted repository which may not be a package source.

`nuget keys refresh -Name NuGet.org`

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

#### Update a trusted repository - 
This scenario should be supported to allow users to update a trusted repository which may not be a package source.

`nuget keys <remove|list|remove|enable|disable> -Name NuGet.org`
<br/>


## Open Questions

* Should we store the certificate expiry?  
_No, since we do not automatically refresh the keys._  

* Should we store the Hash algorithm name or OID?

* Is `nuget keys` confusing with `nuget setApiKey`?

* Should the config element be `packageSourceTrustInformation` or `repositoryTrustInformation`?  
_repository is a new term from nuget.config point of view. Credentials are stored using `packageSourceCredentials`_  

* Should we add a package source as a trusted repository by default?  
_More user friendly as users don't need to add a `--trusted` switch. But a concern is that we should not add trust without user confirmation/action_

* Should we delete repository trust information on source delete?
_Yes. Again we should not implicitly add trust without user confirmation/action. Further, we are leaving a footprint. Users can choose to add the trusted repository as a new action._

* Should we add client policies as part of this spec?