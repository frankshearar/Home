# Fallback packages folder

Fallback packages folders allow packages to be shared across users and machines to reduce disk space. These folders are treated as fallback folders for the primary global packages folder (``%USERPROFILE%\.nuget\packages``). They differ from package sources in that the package assets will be referenced directly and will not be copied into the user's packages folder.

The concept of a fallback package folder can be thought of as a GAC for nupkgs.

### Goal
The goal of the fallback packages folder is to reduce disk space. Common packages such as *NetStandard.Library* can exist on a network share and multiple users may reference the package assets directly from the share.

### Precedence
Package folders have an order of precedence. During restore and build the user's packages folder should be checked first for a package, then each fallback packages folder in order. Once a package has been found the search stops. Search is based on the id and version of the package only. Where the package was originally downloaded from and other asset information is not considered.

The user's package folder is *always* overrides fallback packages folders.

### Folder format
Fallback package folders must use the NuGet v3 folder format of ``{id-lowercase}/{version-lowercase)/{package contents}``. This is the same format as the user package folder format.

The contents of the folder must contain:

1. The extracted package
1. ``{id-lowercase}.nuspec``
1. ``{id-lowercase}.{version-lowercase}.nupkg.sha512``

The folder may optionally contain the nupkg under: ``{id-lowercase}.{version-lowercase}.nupkg``

This format may be created today using ``nuget.exe add`` or ``nuget.exe init``. It may also be created by copying an existing user packages folder that has been created by ``dotnet restore``.

### Immutable
Fallback packages folders are never modified by NuGet restore. New packages will be installed only to the user's packages folder.

### Defining fallback folders

Fallback folders may be defined using either an environment variable or *NuGet.Config*. If the environment variable is non-empty *NuGet.Config* will be ignored.

##### Environment variable
Environment variable: ``NUGET_FALLBACK_PACKAGES``

The value must contain the full path or paths to the fallback package folders. Multiple paths may be included by setting the value to a ``;`` delimited list.

##### NuGet.Config

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <fallbackPackageFolders>
    <add key="Shared" value="\\server\sharedPackages" />
    <add key="MachineWide" value="e:\machineWide" />
    <add key="Relative" value="..\..\global" />
  </fallbackPackageFolders>
</configuration>
```

Folders may be defined in NuGet.Config using the ``fallbackPackageFolders`` section. Unlike ``packageSources`` only ``add`` is supported here.

Ordering is based on the location of the NuGet.Config file and the order within the file. Entries at the top of the list are used first. Config files nearest to the project are ordered before config files in a higher level parent directory. Machine wide settings are applied last and have the lowest precedence. 

#### Tools
The ``.tools`` folder will be ignored for fallback folders. The lock files for tools may only exist at the user level.

