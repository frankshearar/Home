Status: **Review**

## Issue
Parent spec - [Repository-Signatures](https://github.com/NuGet/Home/wiki/Repository-Signatures)  
Related Spec - [Trusted Sources](https://github.com/NuGet/Home/wiki/%5BSpec%5D-NuGet-Config-schema-changes-to-enable-repository-signatures)

## Problem
As we enable author and repository package signing, we need to enable consumers to be able to control how NuGet package signature revocation check is performed. Further, the information needs to be stored into the user's machine.

## Who is the customer?
All NuGet package consumers.

## Scenarios
Enable package consumers to store NuGet package signature revocation check mode.

## Solution
* Define NuGet package signature revocation check mode.
* Update the schema for the nuget.config file to be able to store NuGet package revocation check mode.
* Define a gesture for users to be able to choose NuGet package revocation check mode.

NuGet package signing client policies have been outlined in the [Repository-Signatures spec](https://github.com/NuGet/Home/wiki/Repository-Signatures#client-policies). This spec proposes schema changes to nuget.config and user gestures.

### Info, warning, errors
* We were unable to check revocation because the revocation server was not reachable due to network issues and signatureRevocationCheck = online

`WARNING: NU3028: The author primary signature's timestamp found a chain building issue: The revocation function was unable to check revocation because the revocation server could not be reached. You can switch to offline revocation check. For more information, visit https://aka.ms/offlinePackageRevocationCheck.`

* We were unable to check revocation because the revocation server was not reachable because signatureRevocationCheck = offline

`WARNING: NU3028: The author primary signature's timestamp found a chain building issue: The revocation function was unable to check revocation because the certificate is not available in the cached certificate revocation list and signatureRevocationCheck has been set to offline mode. For more information, visit https://aka.ms/onlinePackageRevocationCheck.`

### Revocation check location
We should store the selected revocation check mode for the user in a `nuget.config` file as a configuration.

### Revocation check information Schema

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <config>
    <add key="signatureRevocationCheck" value="MODE"/>
  </config>
</configuration>
```
The key and value are case insensitive. 

For example -
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <config>
    <add key="signatureRevocationCheck" value="online" />
  </config>
</configuration>
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <config>
    <add key="signatureRevocationCheck" value="offline" />
  </config>
</configuration>
```

### Revocation check mode gesture
To set the NuGet package revocation check mode, users can use the existing [`nuget config`](https://docs.microsoft.com/en-us/nuget/tools/cli-ref-config) command.
<br/>

#### Set 

`NuGet.exe config -set signatureRevocationCheck=online`

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <config>
    <add key="signatureRevocationCheck" value="online" />
  </config>
</configuration>
```

`NuGet.exe config -set signatureRevocationCheck=offline`

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <config>
    <add key="signatureRevocationCheck" value="offline" />
  </config>
</configuration>
```
<br/>

#### Update 

`NuGet.exe config -set signatureRevocationCheck=offline`

Before -
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <config>
    <add key="signatureRevocationCheck" value="online" />
  </config>
</configuration>
```
After -
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <config>
    <add key="signatureRevocationCheck" value="offline" />
  </config>
</configuration>
```
<br/>

#### Remove 

`NuGet.exe config -set signatureValidationMode=`

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
</configuration>
``` 
<br/>

#### Default Value - 
If `signatureRevocationCheck` is not set then NuGet Client should read that as `online` mode.

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
</configuration>
``` 

The above config should be read as having `signatureRevocationCheck=online`.

<br/>

#### Invalid Value - 
If `signatureRevocationCheck` is set to any value other than the supported modes, then NuGet Client should read that as `online` mode and warn the user with a message requesting them to fix the mode value.

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <config>
    <add key="signatureRevocationCheck" value="RANDOM" />
  </config>
</configuration>
``` 

The above config should be read as having `signatureRevocationCheck=online` and the following message should be shown to the user - 

```
NUxxxx: Invalid signatureRevocationCheck mode found in config file <path>. Defaulting to online mode. Please set it to one of the supported modes by running the nuget config command. 
For more information, visit http://docs.nuget.org/docs/reference/command-line-reference.
```

<br/>

#### Revocation check mode in Visual Studio -
The perf impact of revocation check seems to primarily affect CI/CD scenarios. Hence, having a way to controls this setting through the VS PM UI will be deferred to v2.
<br/>


###  Impact of repository signing to client policies - 

* By default NuGet client should operate in online mode (irrespective of the value of signatureValidationMode) where the client will perform author/repository/signedcms signature verification for packages which contain valid signatures.  
* If a user does not have any package sources then NuGet client should write down nuget.org as a package and trusted source and signatureRevocationCheck mode as online into the user nuget.config file.
* NuGet client should respect any trusted source in user settings and perform complete repository signature verification for any package from those sources.