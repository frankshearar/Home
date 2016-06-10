# Server-side warnings for NuGet client

## Problem

The NuGet client issues warnings an errors as they are encountered. For example a warning is shown when running `nuget pack` and package metadata is invalid. Or an error when `nuget push` fails to authenticate with a remote server.

Currently, NuGet servers can return errors to the NuGet client using standard HTTP error codes. However adding a warning to responses is not possible.

This spec proposes an approach to add server-side warnings support to all HTTP communications.

## Solution

The server can return a NuGet-specific HTTP header, `NuGet-Warning`. When encountered in any HTTP response from a NuGet server, the NuGet client will display a warning message.

_Note the `NuGet-Warning` header is not mandatory. When not present, no warning messages should be displayed. The header can occur multiple times. For each value of the header, a warning message must be shown_

Here's an example where the API key of the user is about to expire. The server responded with a `NuGet-Warning` header that held the full warning text:

![](https://cloud.githubusercontent.com/assets/880728/15927309/233633da-2e41-11e6-8a2d-191a4c750d51.png)
