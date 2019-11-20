# UI Delay during ISettings initialization

* Status: **Draft**
* Author(s): [Kartheek Penagamuri](https://github.com/kartheekp-ms)

## Issue

[8675](https://github.com/nuget/home/issues/8675) - UI delay while initializing NuGet.Configuration.ISettings type

## Problem Background
VS IDE customers are experiencing UI Delays when NuGet tries to initialize [`NuGet.Configuration.ISettings`](https://github.com/NuGet/NuGet.Client/blob/0c59e87628fbcbd158162ebb61638ce20e0dc75c/src/NuGet.Clients/NuGet.PackageManagement.VisualStudio/IDE/ExtensibleSourceRepositoryProvider.cs#L63) (Lazy type) in constructor of [`VsPackageSourceProvider`](https://github.com/NuGet/NuGet.Client/blob/0c59e87628fbcbd158162ebb61638ce20e0dc75c/src/NuGet.Clients/NuGet.VisualStudio.Implementation/Extensibility/VsPackageSourceProvider.cs#L25) type on the main UI thread.

## Who are the customers

All VS IDE customers

## Solution

[Andrew Arnott](https://github.com/AArnott) mentioned following important points in an offline conversation.
* MEF parts are not supposed to have any thread affinity, so moving the realization of exports (all the disk I/O from assembly loads, JIT time) and other non-UI code to a background thread could dramatically reduce the UI delay.
* Moving the heavyweight code that’s in the MEF activation path out of that path and into other methods that can be made asynchronous.

### Short-Term solution
* 

### Long-Term solution
* 

![Current references](https://user-images.githubusercontent.com/52756182/69196015-205bdf80-0ae2-11ea-9556-32376ccd151e.png)

## Implementation