* Status: **Reviewing**
* Author(s): [Nikolche Kolev](https://github.com/nkolev92), [Alex Mullans](https://github.com/alexmullans)

## Issue

NuGet cross plat authentication plugin [#6486](https://github.com/NuGet/Home/issues/6486)

## Problem

Currently NuGet has a simple plugin model that's used for authentication against protected feeds. However, it's only available in VS and NuGet.exe. 
* Consumers need to be able to restore against protected feeds cross-plat in msbuild/dotnet.exe. 
* Additionally, we want to establish trust between the process running restore and the credential provider before running it.

## Who are the customers?

The customers are all users of authenticated feeds like VS Team Services (Package Management), MyGet, etc.

## Solution

To facilitate the cross-plat authentication NuGet will extend the extensibility plugin model introduced in version 4.3 as described in the NuGet Package Download Plugin  [spec](https://github.com/NuGet/Home/wiki/NuGet-Package-Download-Plugin).
The protocol established there is versioned and will be extended to further fit the needs of the authentication plugin.

The new protocol version for the Plugin will be **2.0.0**.
Each of the **new requirements** below are **required only** for version **2.0.0** of plugins. 
Every requirement/behavior specified by the previous spec still stands for version **2.0.0** plugins, unless called out specifically.

The high level overview how the plugin integration will work in the authentication case:

1. NuGet discovers available plugins.
2. Due to performance considerations, NuGet will only launch a plugin if/when it has to (401 response from the server). 
3. NuGet will iterate over the plugins in a priority order where applicable, and launch each one.
4. NuGet will ask the plugin for credentials. The plugin should provide valid credentials ONLY if it can help authenticate the user for the feed. 
5. NuGet client tools will shutdown plugins when they are no longer needed. [Read more - Plugin life-cycle ](#life-cycle-management-of-plugins)

## General Plugin Requirements

Each Version **2.0.0** plugin must meet the requirements previous specified in the NuGet Package Download in addition to some new ones warranted by the current use-case.

For simplicity, the requirements will be duplicated from the other spec and altered as required.

- On Windows we require that plugins have a valid Authenticode signature. On non-Windows platforms we will have additional requirements defined at a later point.[Task](#cross-plat-trust-verification)
- Support stateless launching under the current security context of NuGet client tools. For example, NuGet client tools will not perform elevation or additional initialization outside of the plugin protocol described later.
- Support both interactive and non-interactive scenarios. Some operations are allowed to be interactive, as defined below.
- Adhere to the negotiated plugin protocol version.
- Respond to all requests within a reasonable time period.
- Honor cancellation requests for any in-progress operation.

## Supported operations

In addition to the Package Download operation, the new version of the plugin will support Authentication. The plugin in this case is only responsible for:

- Providing valid token to NuGet, that Nuget Client Tools will use for all https communication to the given source.

This token will be sent to the server by NuGet Client Tools in the basic auth form, similar to the previous credential provider workflow.

## Plugin Installation and discovery

Currently NuGet the 1st iteration of extensibility plugins are discovered via an environment variable NUGET_PLUGIN_PATHS, priority preserved.
This behavior will remain backwards compatible.
Additionally, a convention based plugin discovery will be added.
**No ordering** is defined for plugins discovered based on the later defined conventions.

Due to the fact that NuGet runs under both .NET framework and .NET Core, there will be small differences in the discovery.
The plugins will be discovered as follows:

1. An environment variable NUGET_PLUGIN_PATHS, priority reserved of the plugins reserved. If the NUGET_PLUGIN_PATHS environment variable is set, it overrides the convention based plugin discovery.

    The environment variable should contain a full path to the executable, exe in the .NET Framework case and dll in the .NET Core case. It's at the user's discretion to make sure that the plugins are executable under that runtime.
2. User-location - The NuGet Home location - %UserProfile%/.nuget/plugins/

    .NET Core based plugins should be installed in:
    > %UserProfile%/.nuget/plugins/netcore

    .NET Framework based plugins should be installed in:
    > %UserProfile%/.nuget/plugins/netfx

    Each plugin will be installed in it's own folder.

    The plugin entry point will be the name of the installed folder, with the .dll extensions for .NET Core, and .exe extension for .NET Framework.
    To make it more performant, NuGet will assume that the folder name and assembly name always have the same casing, regardless of the operating systems' case sensitivity. 

    Example:

    ```
    .nuget
        plugins
            netfx
                myFeedCredentialProvider
                    myFeedCredentialProvider.exe
                    nuget.protocol.dll
            netcore
                myFeedCredentialProvider
                    myFeedCredentialProvider.dll
                    nuget.protocol.dll
    ``` 

    There's a [gap](issues-with-the-discovery-of-the-user-location-plugins) in this approach.

3. Predetermined location in Visual Studio and dotnet (relative to MsBuild.exe)

    The fixed location would be a folder such as, nugetplugins and will follow the same directory rules as listed under 2.

All plugins should be self-contained, and install all their dependencies in their respective folders.

### Issues with the discovery of the user-location plugins

Different versions of nuget.exe/dotnet.exe could be using the same plugins.
The obvious issue arises when an older client tries to execute a newer plugin.
If a dotnet.exe under the 2.x runtime, tries to launch a 3.x plugin.
A potential approach would involve adding the target framework under which the plugin has been built.
As this a corner-case scenario that's easily resolvable, the current approach is that we go with the cleaner approach of not including any compatibility checks.

## Cross-Plat instantiation of Plugins

On cross-plat, we will rely on the fact that plugins can be executed under the dotnet runtime.
The dll discovered will be run with:

```
dotnet exec plugin.dll
```

## Plugin Interaction

Unlike Package Download, Authentication should be a package agnostic operation.
NuGet will reuse the same operation claims message as before, but query without any parameters.

For simplicity the following rules should be followed by the plugins.
On the request for source specific operation, the plugin should not return operations such as Authentication.
On the request for source agnostic operations, the plugin should not return operations such as PackageDownload.
It's the consumers' job to make sure it doesn't attempt to delegate an unsupported operation to a plugin.

## Plugin Protocol

All the protocol rules defined by **Version 1.0.0** stand for **Version 2.0.0**.

## Plugin Protocol Version Negotiation

The version negotiation stays the same as defined in Version 1.0.0.

## Application messages

The following message will be amended for version 2.0.0 of the plugin.

1. Get Operation Claims

* Request direction:  NuGet -> plugin
    * The request will contain:
        * the service index.json for a package source
        * the package source repository location
    * A response will contain:
        * a response code indicating the outcome of the operation
        * an enumerable of supported operations if the operation was successful.  If a plugin does not support the package source, the plugin must return an empty set of supported operations.

    If the service index and package source are null, then the plugin can answer with authentication.

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
    * List of Auth Types
    * MessageResponseCode

## Proxy support for the plugins

The NuGet client has basic proxy support. [More info](https://docs.microsoft.com/en-us/nuget/reference/nuget-config-file)
The first credential plugin iteration implementation did not communicate any proxy information.
In the new implementation, the client will make sure that the proxy information is passed to the by sending a "SetCredentials" message. 
[More info](https://github.com/NuGet/Home/wiki/NuGet-Package-Download-Plugin#application-messages) about the signature of said message. 

## How does Visual Studio work with the plugin

In the previous credential plugin implementation, the experience in Visual Studio and NuGet.exe was different.
NuGet.exe used the plugin, while in Visual Studio, authentication was done by relying on an extra VSIX that brings in a Credential provider API.

For performance considerations we will keep the same approach for Visual Studio.
As a fallback, NuGet will use the credential plugins, if no other Credential Providers in Visual Studio can satisfy the scenario.

## How does NuGet.exe work with the plugin

In the old plugin model, NuGet.exe discovered the plugins next to it's executable location.
Now NuGet.exe will be able to depend on the plugins the same way dotnet.exe, msbuild.exe and Visual Studio do.
That means, that the version of MsBuild that NuGet.exe uses (latest by default unless specifically specified) will dictate which built in plugins NuGet.exe discovers.

## How do dotnet.exe/MSBuild.exe work with the plugin

MSBuild.exe and dotnet.exe do not prompt by default like Visual Studio and NuGet.exe do.
Because of that, the authentication strategy for dotnet.exe and MsBuild.exe will be device flow.
The default behavior is that no authentication will happen in the plugin unless an interactivity flag is being passed to the plugin.
Whenever NuGet cannot authenticate, a clear error message suggesting that they might need to pass an extra flag will be displayed.
When the flag is passed, the build/restore will block and the user will be provided with instructions on how to complete the authentication.
Once that's completed, the action will resume.

### Life-cycle management of plugins

The plugins currently control different operations in NuGet. Because all of these are independent of each other, it is very hard to understand whether a plugin is still needed.
We additionally want to avoid starting a new process too often, so because of that, NuGet will manage the plugin lifetime with idleness.
In practice, idleness is only relevant in Visual Studio. In the command-line scenarios, the plugin will die when the NuGet process ends.

## Open work items

### Installation directory for the built in plugin

Ideally, a similar pattern will be followed in dotnet and Visual Studio. Only requirement on NuGet side is that it's a static location relative to MsBuild. 

### Cross-plat trust verification

Currently, the verification will happen on Windows and fail on cross-plat.
We need to come up with a verification mechanism cross-plat.

### Avoid instantiating the plugins frequently

Currently, the approach with the plugin discovery is to load each plugin when there’s an operation that needs delegated (PackageDownload).
This is acceptable in the current implementation as there are not a lot of plugins.
In the next implementation, there will be a baked in plugin that will always be discovered.
Even in the case when there’s no need for a credential plugin, whenever a package download occurs, we’d start the baked in credential plugin to query for operations.
This adds a lot of overhead.
The approach here is to use a caching strategy similar to the http cache.