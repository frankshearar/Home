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
2.	NuGet client tools launch each plugin and query plugin claims (e.g.:  "I handle package download") per package source.
3.	For each package download and related action, NuGet client tools send a request to the first plugin that claims to handle the operation.
4.	Plugins will perform the requested operation and respond to NuGet client tools with success/failure details.
5.	NuGet client tools will shutdown plugins when they are no longer needed.

To facilitate future investments in cross-platform support, the target platform for these changes will be .NET Core.
Plugins are optional mechanism for extending NuGet client tool functionality and not required for any current default package download behaviors.

Since VSTS only requires V3 download support, only V3 download support will be implemented.  Plugins will not be able to override V2 package download.

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
* Downloading a package to a location indicated by NuGet client tools.

### Plugin Discovery
NuGet tools will discover plugins via a NUGET_PLUGIN_PATHS environment variable.  The environment variable value will be parsed as a semicolon-delimited list of file paths to plugin executables.  The order of plugin file paths is significant.  The first plugin to claim support for an operation and package source will be the sole delegate for that operation and package source.

Empty paths and paths consisting only of whitespace will be ignored.  A path which cannot be resolved to a plugin executable will be reported with a warning.  Relative paths are not allowed.

Plugins must be code signed.  Unsigned or untrusted plugins will be reported with a warning and skipped.  .NET Core does not have built-in support for code signature verification.  On Windows platforms Authenticode signatures will be verified by PInvoking WinVerifyTrust(...).  Other platforms are not supported, and specifying a plugin on other platforms will fail the overall operation with error detail reported to the user.

### Plugin Interaction
NuGet client tools and plugins will communicate with JSON over standard streams (stdin, stdout, stderr).  All data must be UTF-8 encoded.

NuGet client tools will launch a plugin with the argument "-Plugin".  In case a user directly launches a plugin executable without this argument, the plugin can give an informative message instead of waiting for a protocol handshake.

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

The protocol will have a 5-second response timeout default.  If a request does not receive a response within this timeout period, it should result in connection termination.  A new response timeout default may be established with an application-level request/response pair.  However, the 5-second timeout default will remain in effect until a new response timeout default is accepted.

During protocol version negotiation, only the following message types are allowed:  request, response, fault.  For example, progress messages cannot be used during the handshake phase to reset the 5-second response timeout default.

### Application Messages
The following messages are required for supporting the overall download package operation:

1.  Close
    * Request direction:  NuGet -> plugin
    * The request will contain no payload
    * No response is expected.  The proper response is for the plugin process to promptly exit.

2.  Copy files in package
    * Request direction:  NuGet -> plugin
    * The request will contain:
        * the package ID and version
        * the package source repository location
        * destination directory path
        * an enumerable of files in the package to be copied to the destination directory path
    * A response will contain:
        * a response code indicating the outcome of the operation
        * an enumerable of full paths for copied files in the destination directory if the operation was successful

3.  Copy package file (.nupkg)
    * Request direction:  NuGet -> plugin
    * The request will contain:
        * the package ID and version
        * the package source repository location
        * the destination file path
    * A response will contain:
        * a response code indicating the outcome of the operation

4.  Get credentials
    * Request direction:  plugin -> NuGet
    * The request will contain:
        * the package source repository location
        * the HTTP status code obtained from the package source repository using current credentials
    * A response will contain:
        * a response code indicating the outcome of the operation
        * a username, if available
        * a password, if available

5.  Get files in package
    * Request direction:  NuGet -> plugin
    * The request will contain:
        * the package ID and version
        * the package source repository location
    * A response will contain:
        * a response code indicating the outcome of the operation
        * an enumerable of file paths in the package if the operation was successful

6.  Get operation claims
    * Request direction:  NuGet -> plugin
    * The request will contain:
        * the service index.json for a package source
        * the package source repository location
    * A response will contain:
        * a response code indicating the outcome of the operation
        * an enumerable of supported operations (e.g.:  package download) if the operation was successful.  If a plugin does not support the package source, the plugin must return an empty set of supported operations.

7.  Get package hash
    * Request direction:  NuGet -> plugin
    * The request will contain:
        * the package ID and version
        * the package source repository location
        * the hash algorithm
    * A response will contain:
        * a response code indicating the outcome of the operation
        * a package file hash using the requested hash algorithm if the operation was successful

8.  Get package versions
    * Request direction:  NuGet -> plugin
    * The request will contain:
        * the package ID
        * the package source repository location
    * A response will contain:
        * a response code indicating the outcome of the operation
        * an enumerable of package versions if the operation was successful

9.  Handshake
    * Request direction:  NuGet <-> plugin
    * The request will contain:
        * the current plugin protocol version
        * the minimum supported plugin protocol version
    * A response will contain:
        * a response code indicating the outcome of the operation
        * the negotiated protocol version if the operation was successful.  A failure will result in termination of the plugin.

10.  Initialize
     * Request direction:  NuGet -> plugin
     * The request will contain:
         * the NuGet client tool version
         * the NuGet client tool effective language.  This takes into consideration the ForceEnglishOutput setting, if used.
         * the default request timeout, which supersedes the protocol default.
     * A response will contain:
         * a response code indicating the outcome of the operation.  A failure will result in termination of the plugin.

11.  Log
     * Request direction:  plugin -> NuGet
     * The request will contain:
         * the log level for the request
         * a message to log
     * A response will contain:
         * a response code indicating the outcome of the operation.

12.  Monitor NuGet process exit
     * Request direction:  NuGet -> plugin
     * The request will contain:
         * the NuGet process ID
     * A response will contain:
         * a response code indicating the outcome of the operation.

13.  Prefetch package
     * Request direction:  NuGet -> plugin
     * The request will contain:
         * the package ID and version
         * the package source repository location
     * A response will contain:
         * a response code indicating the outcome of the operation

14.  Set credentials
     * Request direction:  NuGet -> plugin
     * The request will contain:
         * the package source repository location
         * the last known package source username, if available
         * the last known package source password, if available
         * the last known proxy username, if available
         * the last known proxy password, if available
     * A response will contain:
         * a response code indicating the outcome of the operation

15.  Set log level
     * Request direction:  NuGet -> plugin
     * The request will contain:
         * the default log level
     * A response will contain:
         * a response code indicating the outcome of the operation


### Required work
* Add a PluginResourceProvider resource provider and a PluginResource resource to discover and interact with plugins, respectively.
* Add a DownloadResourcePluginProvider resource provider and a DownloadResourcePlugin resource to delegate package downloads in packages.config scenarios.  This resource provider will have higher priority than the current default resource provider (DownloadResourceV3Provider).
* Add PluginFindPackageByIdResourceProvider and PluginFindPackageByIdResource to delegate package downloads in project.json scenarios.  This resource provider will have higher priority than the current default resource provider (HttpFileSystemBasedFindPackageByIdResourceProvider).
* Add PluginPackageReader that subclasses PackageReaderBase and delegates package read operations to a plugin.
* Add bidirectional IPC API’s for use by both NuGet client tools and plugins.