# UI Delay during ISettings initialization

* Status: **Draft**
* Author(s): [Kartheek Penagamuri](https://github.com/kartheekp-ms)

## Issue

[8675](https://github.com/nuget/home/issues/8675) - UI delay while initializing NuGet.Configuration.ISettings type

## Problem Background
VS IDE customer experiences UI Delay since NuGet tries to initialize `NuGet.Configuration.ISettings` (Lazy type) in the constructor of `VsPackageSourceProvider` type on the main UI thread. 

```[AArnott](https://github.com/AArnott) mentioned following important points in an offline conversation.
* MEF parts are not supposed to have any thread affinity, so moving the realization of exports to a background thread can move all the disk I/O from assembly loads, JIT time, and other non-UI code to a background thread and could dramatically reduce the UI delay.
* Moving the heavyweight code that’s in the MEF activation path out of that path and into other methods that can be made asynchronous.```

## Who are the customers

All VS IDE customers

## Solution

## Implementation