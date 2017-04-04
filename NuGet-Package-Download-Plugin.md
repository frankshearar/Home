# NuGet Package Download Plugin

## Issue
[#4913](https://github.com/NuGet/Home/issues/4913)
## Problem
VSTS needs to be able to override the default behaviors for download package with a custom implementation to have greater control over package transmission.
## Who is the customer?
The primary customers are engineers working with large binaries within Microsoft; however, this is a public extension point.
## Solution
To facilitate custom download implementations, NuGet tools will expose a new extensibility model for overriding package downloads.  A “plugin” will provide a custom implementation for package download.  Plugin design will borrow some design choices from the existing credential provider extensibility model.  The high-level process will be:
1.	NuGet client tools discover available plugins.
2.	NuGet client tools launch each plugin and query plugin claims (e.g.:  “I handle package download”) per package source.  The first plugin to claim support for an operation and package source will be the sole delegate for that operation and package source.
3.	For each package download and related action, NuGet client tools send a request to the first plugin that claims to handle the operation.
4.	Plugins will perform the requested operation and respond to NuGet client tools with success/failure details.
5.	NuGet client tools will shutdown plugins when they are no longer needed.

To facilitate future investments in cross-platform support, the target platform for these changes will be .NET Core.
Plugins are optional mechanism for extending NuGet client tool functionality and not required for any current default package download behaviors.

# Plugin Discovery
NuGet tools will discover plugins via a NUGET_PLUGIN_PATHS environment variable.  The environment variable value will be parsed as a semicolon-delimited list of file paths to plugin executables.

Empty paths and paths consisting only of whitespace will be ignored.  A path which cannot be resolved to a plugin executable will be reported with a warning.  Relative paths will be supported and will be resolved relative to the executing NuGet tool process.

Plugins must be code signed.  Unsigned or untrusted plugins will be reported with a warning and skipped.
.NET Core does not have built-in support for code signature verification.  On Windows platforms Authenticode signatures will be verified by PInvoking WinVerifyTrust(...).  Other platforms will throw a PlatformNotSupportedException.

# Plugin Interaction
NuGet client tools and plugins will communicate with JSON over standard streams (stdin, stdout, stderr).

<span style="background: yellow">TODO:  detail the JSON protocol.</span>

NuGet client tools will query a plugin’s supported operations by passing in the service index for a NuGet source.  A plugin may use the service index to check for the presence of supported service types.  Plugins providing a custom implementation for package download is responsible for all the following:
* Downloading a package to a location indicated by NuGet client tools
* Providing the package’s .nuspec file.
    * If a plugin downloaded a standard .nupkg file, NuGet client tools will obtain the .nuspec file from the .nupkg file as it currently does today.
    * If a plugin downloaded a package in a nonstandard format, the plugin is responsible for providing the .nuspec file to NuGet client tools.
* Honoring cancellation requests for any in-progress operations.

## Required work
* Add a PluginsResourceProvider resource provider and a PluginsResource resource to discover and interact with plugins, respectively.
* Add a DownloadResourcePluginProvider resource provider and a DownloadResourcePlugin resource to delegate package downloads in packages.config scenarios.  This resource provider will have higher priority than the current default resource provider (DownloadResourceV3Provider).
* Add PluginFindPackageByIdResourceProvider and PluginFindPackageByIdResource to delegate package downloads in project.json scenarios.  This resource provider will have higher priority than the current default resource provider (HttpFileSystemBasedFindPackageByIdResourceProvider).
* Add PluginPackageReader that subclasses PackageReaderBase and delegates package read operations to a plugin.
* Add bidirectiontional IPC API’s for use by both NuGet client tools and plugins.