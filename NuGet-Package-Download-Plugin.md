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
2.	NuGet client tools launch each plugin and query plugin claims (e.g.:  “I handle package download”) per package source.
3.	For each package download and related action, NuGet client tools send a request to the first plugin that claims to handle the operation.
4.	Plugins will perform the requested operation and respond to NuGet client tools with success/failure details.
5.	NuGet client tools will shutdown plugins when they are no longer needed.

To facilitate future investments in cross-platform support, the target platform for these changes will be .NET Core.
Plugins are optional mechanism for extending NuGet client tool functionality and not required for any current default package download behaviors.

### General Plugin Requirements
A plugin must:
* Have a valid, trusted Authenticode signature.
* Support stateless launching under the current security context of NuGet client tools.  For example, NuGet client tools will not perform elevation or additional initialization outside of the plugin protocol described later.
* Be noninteractive.
* Adhere to the negotiated plugin protocol version.
* Respond to all requests within a reasonable time period.
* Honor cancellation requests for any in-progress operation.

### Supported Operations
Package download is the only NuGet operation available for plugins to override.  A plugin providing a custom implementation for package download is responsible for all the following:
* Providing a list of files in a package and a full path for an individual file to be opened with read access.  Request for this information is independent of a package download request.  This information is necessary for dependency graph resolution in project.json scenarios.
* Package download caching
  * Honoring cache hints (e.g.:  NoCache, DirectDownload, etc.) provided by NuGet client tools.  A plugin's cache is independent of any cache maintained by NuGet client tools.
  * Managing the plugin's own cache (e.g.:  location, concurrency, invalidation, purging, etc.).  Plugin cache management is out of scope for this spec.  (At some future time "nuget locals" functionality could be plumbed through to plugins.)
* Downloading a package to a location indicated by NuGet client tools

### Plugin Discovery
NuGet tools will discover plugins via a NUGET_PLUGIN_PATHS environment variable.  The environment variable value will be parsed as a semicolon-delimited list of file paths to plugin executables.  The order of plugin file paths is significant.  The first plugin to claim support for an operation and package source will be the sole delegate for that operation and package source.

Empty paths and paths consisting only of whitespace will be ignored.  A path which cannot be resolved to a plugin executable will be reported with a warning.  Relative paths will be supported and will be resolved relative to the executing NuGet tool process.

Plugins must be code signed.  Unsigned or untrusted plugins will be reported with a warning and skipped.  .NET Core does not have built-in support for code signature verification.  On Windows platforms Authenticode signatures will be verified by PInvoking WinVerifyTrust(...).  Other platforms are not supported, and specifying a plugin on other platforms will fail the overall operation with error detail reported to the user.

### Plugin Interaction
NuGet client tools and plugins will communicate with JSON over standard streams (stdin, stdout, stderr).

NuGet client tools will query a plugin’s supported operations by passing in the service index for a NuGet source.  A plugin may use the service index to check for the presence of supported service types.  

### Plugin Protocol
Version 1.0.0 defines the following rules:

1.  All messages (requests, responses and notifications) must be well-formed JSON.
2.  All messages must have an associated request ID.  This ID is generated for a request message and echoed in all associated response messages and progress notifications.
3.  Every request requires a response.
4.  A response is required within a predefined timeout period; however, progress notifications associated with a request reset the timeout timer.
5.  A response must either be a fault or completion message.
6.  A fault response should be interpreted by the receiver as an exception in the sender.
7.  Requests must terminate "quickly" on a cancellation request.

Protocol errors include:
* A malformed response or request.
* A response without an associated request.
* A progress notification received for a request after the response for the request has been received.
* A response or progress notification not being received within an predefined timeout period.

All messages must have the following common fields:

1.  Request ID:  a string containing the unique identifier for a request
2.  Message type:  a string indicating if the message type (e.g.:  "request", "response", "progress", "fault" or "cancel").
3.  Method:  a string representing the request (e.g.:  "handshake", "downloadPackage", etc.)

### Plugin Protocol Version Negotiation
Both parties will attempt to negotiate a common protocol version by sending to the other party a connection request message indicating their current protocol version as well as their minimum supported protocol version, in case a party maintains support for older protocol versions for backwards compatibility.  All version strings must be [SemVer 2.0.0](http://semver.org/spec/v2.0.0.html)-compliant.

The connection request will communicate both a protocol version as well as a minimum protocol version for optional backwards compatibility.

Failure to negotiate a common protocol version must result in termination of the plugin.

### Application Messages
The following messages are required for supporting the overall download package operation:

1.  Initialize
    * The request message will contain:
      * NuGet client tool version
      * NuGet client tool effective language.  This takes into consideration the ForceEnglishOutput setting, if used.
      * Verbosity level
      * Response timeout
    * A response failure will result in termination of the plugin.

2.  Operation claims query
    * The request message will contain the service index.json for a package source.
    * The response message will contain supported operations (e.g.:  package download).  If a plugin does not support the package source, the plugin must return an empty set of supported operations.

3.  Get files
    * The request will contain:
	* package source credentials
	* proxy and proxy credentials
	* package ID
	* package version
    * The response will contain a list of files in the package.

3.  Get file path
    * The request will contain:
      * package source credentials
      * proxy and proxy credentials
      * package ID
      * package version
      * the file path within the package
    * The response will contain a readable file path for the specified file.

4.  Package download
    * The request will contain:
    * package source credentials
      * proxy and proxy credentials
      * package ID
      * package version
      * PackageSaveMode
      * XmlDocFileSaveMode
      * installation location
      * hash file path
      * hash algorithm (e.g.:  SHA-512)
    * The response will indicate success or failure.  Success requires the following:
      * the package has been extracted to the specified installation location
      * a hash file has been written to the specified location.  This hash file must not exist until all the other files are completely available in the destination.
      * the hash file contents is the SHA-512 hash for the package

5.  Shutdown
    * The request will contain no additional data.
    * The response will return acknowledgement.

### Required work
* Add a PluginsResourceProvider resource provider and a PluginsResource resource to discover and interact with plugins, respectively.
* Add a DownloadResourcePluginProvider resource provider and a DownloadResourcePlugin resource to delegate package downloads in packages.config scenarios.  This resource provider will have higher priority than the current default resource provider (DownloadResourceV3Provider).
* Add PluginFindPackageByIdResourceProvider and PluginFindPackageByIdResource to delegate package downloads in project.json scenarios.  This resource provider will have higher priority than the current default resource provider (HttpFileSystemBasedFindPackageByIdResourceProvider).
* Add PluginPackageReader that subclasses PackageReaderBase and delegates package read operations to a plugin.
* Add bidirectional IPC API’s for use by both NuGet client tools and plugins.