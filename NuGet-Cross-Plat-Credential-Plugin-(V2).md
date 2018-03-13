## NuGet Authentication Plugin

### Issue
[6642](https://github.com/NuGet/Home/issues/6642) - NuGet credential plugin V2

[6410](https://github.com/NuGet/Home/issues/6410) - msbuild /restore support for all NuGet use cases

### Problem
Currently NuGet has a simple plugin model that's used for authentication against protected feeds. 
The problem is currently that's only available in VS and NuGet.exe. 
Consumers need to be able to restore against protected feeds cross-plat in msbuild/dotnet.exe. 
Additionally we want to establish trust between the process running restore and the credential provider before running it.

### Who is the customers?
The customers are all users of protected feeds. That includes the majority of VSTS/MyGet users. 

### Solution

To facilitate the cross-plat authentication NuGet will extend the existing extensbility plugin model as described in the NuGet Package Download Plugin [spec](https://github.com/NuGet/Home/wiki/NuGet-Package-Download-Plugin). 
The protocol established there is versioned and will be extended to further fit the needs of the authentication plugin. 

The new protocol version for the Plugin will be **2.0.0**. 
Each of the **new requirements** below are **required only** for version **2.0.0** of plugins. 
**Every** requirement/behavior specified by the previous spec, still stands for 2.0.0 plugins, unless called out specifically.

The high level overview how the plugin integration will work in the authentication case:
1. NuGet discovers available plugins. [Task](#plugin-discovery-cross-plat-for-credential-providers-baked-into-msbuild)
2. Due to performance considerations, NuGet will only launch a plugin if/when it has to (401 response from the server). 
3. NuGet will iterate over the plugins in a priority order, and launch each one. 
4. NuGet will ask the plugin for credentials. The plugin should provide valid credentials ONLY if it can help authenticate the user for the feed. 
5. NuGet client tools will shutdown plugins when they are no longer needed. [Task](#life-cycle-management-of-plugins)

### General Plugin Requirements
Each plugin must meet the requirements previous specified in the NuGet Package Download in addition so some new ones warranted by the current use-case. 

For simplicity, the requirements will be duplicated from the other spec and altered as required.

- ~~Have a valid, trusted Authenticode signature.~~ Have a valid, trusted Authenticode signature, additionally fit the requirements for Cross-Plat trust verification. (Task)[#cross-plat-trust-verification]
- Support stateless launching under the current security context of NuGet client tools. For example, NuGet client tools will not perform elevation or additional initialization outside of the plugin protocol described later.
- ~~Be noninteractive.~~ Support both interactive and non-interactive scenarios. Some messages are allowed to be interactive, as defined below.
- Adhere to the negotiated plugin protocol version.
- Respond to all requests within a reasonable time period.
- Honor cancellation requests for any in-progress operation.

### Supported operations
In addition to the Package Download operation, the new version of the plugin will support Authentication. 
The plugin in this case is only responsible for:
- Providing valid token to NuGet, that Nuget Client Tools will use for all https communication to the given source. 
This token will be sent to the server by NuGet Client Tools in the basic auth form, similar to the previous credential provider workflow.

### Plugin Discovery

Currently NuGet the 2nd iteration of extensibility plugins are discovered via an environment variable NUGET_PLUGIN_PATHS, priority preserved. 
Additionally, we will add support for discovering credential providers brought in by MSBuild itself. [Task](#plugin-discovery-cross-plat-for-credential-providers-baked-into-msbuild)

### Plugin Interaction. 
NuGet will launch a plugin with the "-Plugin". 
Unlike Package Download, Authentication should be a package agnostic operation.
NuGet Client tools will query for a plugin's supported operations in 2 different ways. 
First one, is as described by the previous spec. 
Second one, will be a message for source-agnostic operations. 
For simplicity the same enum will be used for operations, but the following rules should be followed by the plugins.

On the request for source specific operation, the plugin should not return operations such as Authentication. 
On the request for source agnostic operations, the plugin should not return operations such as PackageDownload.
It's the consumers' job to make sure it doesn't attempt to delegate an unsupported operation to a plugin. 

### Plugin Protocol
All the protocol rules defined by **Version 1.0.0** stand for **Version 2.0.0**.

### Plugin Protocol Version Negotiation 
The version negotiation stays the same as defined in Version 1.0.0. 

### Application messages

The following additional messages are required for version 2.0.0 of the plugin. 
1. Get Source Agnostic Operations
* Request direction: NuGet -> plugin
* The request will be empty
* A response will contain
    * List of supported source agnostic operation claims

The following additional messages are required to support the authentication operation. 

2. Authentication Credentials 
* Request direction: NuGet -> plugin
* The request will contain:
    * Uri
    * isRetry
    * NonInteractive
* A response will contain
    * Username
    * Password
    * Message
    * List of Auth Types (TODO NK - currently only basic)

### Open work items

#### Review current proposal with team
#### Cross-plat trust verification
- Currently the verification will happen on full framework and fail on cross-plat. For testing purposes, the bits delivered to VSTS/MSBuild team to test their implementation of the plugin will avoid verifying cross-plat. 
#### Plugin Discovery cross-plat for credential providers baked into MSBuild
- Currently, the only way to specify plugins is using environment variables. We need to think of a way to load in baked-in credentials provider. The experience should be, user installs latest VS, and authentication **just works**
We also need to call out the priority order for these plugins. 
#### Life-cycle management of plugins 
Currently there's only 1 type of plugin, and it is being disposed on idle by NuGet, or the plugin shuts itself down when it discovers that the NuGet process has exited (communicated during the initial negotiation stage). 
We need to understand how to correctly manage these plugins, and make sure there aren't any dangling processes. 
The ideal scenario is NuGet shuts down the plugin when it doesn't need it, but that's very difficult to be done as a general thing. Reference counting is an option. The Authentication plugins will only be launched when needed, can also be discarded there. Problem is what happens when a plugin is both. 