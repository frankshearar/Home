Status: **Incubation**

## Issue
Parent spec - [Repository-Signatures](https://github.com/NuGet/Home/wiki/Repository-Signatures)
Related Spec - [Trusted Sources](https://github.com/NuGet/Home/wiki/Spec:-NuGet-Config-schema-changes-to-enable-repository-signatures)

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
<br/>

## Open Questions

* Rollout plans - 
1. We should default to Dev mode starting VS 15.7. At this point in time if no client policy is set, we should take that as Dev mode.
2. We should add an option in our Visual Studio UI to allow users to change the client policy.
3. At some point in future, when we are ready to change default to Secure mode, we should start warning users on launching Visual Studio that they are using NuGet in an insecure mode. At the same time any new installations should automatically set client policy to Secure mode.
4. If a user changes their client policy to Secure mode, we should evaluate all of their sources and warn if any V3 source is not marked as trusted and warn if there is no trusted source available.