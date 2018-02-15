Status: **In Progress**

## Issue
Parent spec - [Repository-Signatures](https://github.com/NuGet/Home/wiki/Repository-Signatures)  
Related Spec - [Trusted Sources](https://github.com/NuGet/Home/wiki/%5BSpec%5D-NuGet-Config-schema-changes-to-enable-repository-signatures)

## Problem
Once we enable repository package signing, we need to enable consumers to be able to control the NuGet client security policy. Further, the information needs to be stored into the users machine.

## Who is the customer?
All NuGet package consumers.

## Scenarios
Enable package consumers to store repository NuGet client security policy.

## Solution
* Define NuGet client security policies.
* Update the schema for nuget.config file to be able to store NuGet client security policy.
* Define a gesture for users to be able to choose NuGet client security policy.

Client policies have been outlined in the [Repository-Signatures spec](https://github.com/NuGet/Home/wiki/Repository-Signatures#client-policies). This spec proposes schema changes to nuget.config and user gestures. Further, the spec outlines a rollout plan to the default mode for NuGet clients.

### Client Policy Information Location
We should store the selected client policy for the user in a `nuget.config` file as a configuration.

### Client Policy Information Schema
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <config>
    <add key="signatureValidationMode" value="MODE" />
  </config>
</configuration>
```

For example -
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <config>
    <add key="signatureValidationMode" value="dev" />
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
    <add key="signatureValidationMode" value="secure" />
  </config>
</configuration>
```

### Client Policy Information Gesture
To set the nuget client security policy, users can use the existing [`nuget config`](https://docs.microsoft.com/en-us/nuget/tools/cli-ref-config) command. All of the operations are performed on `%AppData%\NuGet\NuGet.config` by default. Users can control the config file using the `-configFile` parameter.
<br/>
<br/>

#### Set 

`NuGet.exe config -set signatureValidationMode=dev`

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <config>
    <add key="signatureValidationMode" value="dev" />
  </config>
</configuration>
```

`NuGet.exe config -set signatureValidationMode=secure`

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <config>
    <add key="signatureValidationMode" value="secure" />
  </config>
</configuration>
```
<br/>

#### Update 

`NuGet.exe config -set signatureValidationMode=secure`

Before -
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <config>
    <add key="signatureValidationMode" value="dev" />
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
    <add key="signatureValidationMode" value="secure" />
  </config>
</configuration>
```
<br/>

#### Clear 

`NuGet.exe config -set signatureValidationMode=""`

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="NuGet.Org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
</configuration>
``` 
<br/>

#### Client Policy in Visual Studio -
We should add support for the following in Visual Studio NuGet options control - 

* Add a drop down menu to enable users to choose a security policy -  
![](https://github.com/NuGet/Home/blob/dev/resources/signing/client%20policy%20selection.png)
<br/>


### Source validation on mode change - 
If the user changes their mode to strict/secure then NuGet client should validate all the available sources as following - 
1. Get all package sources.
2. Get all trusted sources.
3. Filter all package sources to ones which support V3.
4. Check if any of the V3 sources advertise that all of their packages are signed.
5. If any source which is signing all packages is not a trusted source then warn user to trust the users via one of the following gestures -  
a. `nuget sources update -Name <source> -Trust`  
b. Mark source as trusted via Visual Studio UI.


###  Impact of repository signing to client policies - 

#### Dev mode -
* By default NuGet client should operate in dev mode where the client will perform author/repository/signedcms signature verification for packages which contain a valid signatures.  
* If a user does not have any package sources then NuGet client should write down nuget.org as a package and trusted source and signatureValidationMode as dev into the user nuget.config file.
* NuGet client should respect any trusted source in user settings and perform complete repository signature verification for any package from those sources.

#### Secure mode -
* In secure mode NuGet client will only allow packages signed by a list of trusted sources or authors along with all the constraints of dev mode.
* If a user does not have any package sources then NuGet client should write down nuget.org as a package and trusted source and signatureValidationMode as dev into the user nuget.config file.
* If a package is signed by an author or source that is not trusted, then the operation should fail with an error.

#### Changing of modes - 
1. Starting in 4.7, NuGet will operate in dev mode for all users.
2. In a following release NuGet client will allow users to change their security policy as proposed in this spec.
3. If a user changes their mode to secure/strict then we should validate all package sources as described above.
4. If, at some point in future, the default mode needs to be changed from dev to strict/secure, then it should be done only for new installations and with a new major version release as this will be a breaking change.

### Timeline of NuGet.Org package signing and NuGet client policies- 
1. In 4.7 release, NuGet client should have support for verifying repository signatures. At this point, NuGet client should perform repository signature verification on a package with valid repository signature.
2. Starting in 4.7 release, if a source (NuGet.org) starts advertising that all of its packages are signed then NuGet client should assert that any package from that source are signed with a valid repository signature by one of the advertised certificates.
3. In the next NuGet client release after NuGet.org has finished signing all packages, the client should assert that all packages from NuGet.org are repository signed by one of the advertised certificates. If the client is unable to reach NuGet.org, then it should use offline values of the last known certificates.