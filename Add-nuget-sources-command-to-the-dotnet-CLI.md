* Status: **Reviewing**
* Author(s): [Rob Relyea](https://github.com/rrelyea)

## Issue

add `dotnet nuget <add|remove|update|disable|enable|list> source` command [#4126](https://github.com/NuGet/Home/issues/4126)

## Problem Background

`NuGet.exe sources <add|remove|enable|disable|update|list>` functionality hasn't been ported to `dotnet.exe` yet. 

## Who are the customers

Package Authors & Package Consumers who are using .NET core. Many of these devs are used to using dotnet.exe as their primary interface to dotnet core development.

## Requirements

* Blend in nicely with dotnet.exe.
* Provide all important functionality.
* Review behavior, because now would be a good time to change/tweak/improve, especially if it is breaking.

## DotNet CLI command strategy

### Usage: dotnet nuget list source [options]

Options:

 -f|--format Applies to the list action. Accepts two values: Detailed (the default) and Short.

 -c|--configfile  The NuGet configuration file. If specified, only the settings from this file will be used. If not specified, the hierarchy of configuration files from the current directory will be used. To learn more about NuGet configuration go to https://docs.microsoft.com/en-us/nuget/consume-packages/configuring-nuget-behavior.
 
 -h|--help        Show help information

Outputs a list of configured sources, including if they are enabled or disabled.

### Usage: dotnet nuget add source [options]

Options:
  -n|--name <name>                Name of the source.
  -s|--source <source>            Path to the package(s) source.

  -u|--username <username>        UserName to be used when connecting to an authenticated source.

  -p|--password <password>        ...

  --store-password-in-clear-text  Enables storing portable package source credentials by disabling password encryption.

  --valid-authentication-types    Comma-separated list of valid authentication types for this source. By default, all authentication types are valid. Example: basic,negotiate

  -c|--configfile                 The NuGet configuration file. If specified, only the settings from this file will be used. If not specified, the hierarchy of configuration files from the current directory will be used. To learn more about NuGet configuration go to https://docs.microsoft.com/en-us/nuget/consume-packages/configuring-nuget-behavior.

  -h|--help                       Show help information

Consider Post-MVP Improvement:
* target first config file found, not just one with PackageSources - [#1589](https://github.com/NuGet/Home/issues/1589)
* if config file is not found, create one.
* tell which config it was added to or removed from
* where do credentials get written down? is that good? should there be another flag to control? (looks like -configfile will write the source and the creds in the file pointed to????)
* consider validating a source on creation... does the directory exist? does the web url exist? if you cannot access the url, do you need to add a password or credprovider?


### Usage: dotnet nuget update source [options]

Options:
  -n|--name <name>                Name of the source.

  -s|--source <source>            Path to the package(s) source.

  -u|--username <username>        UserName to be used when connecting to an authenticated source.

  -p|--password <password>        UserName to be used when connecting to an authenticated source.

  --store-password-in-clear-text  Enables storing portable package source credentials by disabling password encryption.

  --valid-authentication-types    Comma-separated list of valid authentication types for this source. By default, all authentication types are valid. Example: basic,negotiate

  -c|--configfile                 The NuGet configuration file. If specified, only the settings from this file will be used. If not specified, the hierarchy of configuration files from the current directory will be used. To learn more about NuGet configuration go to https://docs.microsoft.com/en-us/nuget/consume-packages/configuring-nuget-behavior.

  -h|--help                       Show help information

If -name param matches existing source, updates all other properties of that source.


### Usage: dotnet nuget remove source [options]

Options:
  -n|--name <name>  Name of the source.

  -c|--configfile   The NuGet configuration file. If specified, only the settings from this file will be used. If not specified, the hierarchy of configuration files from the current directory will be used. To learn more about NuGet configuration go to https://docs.microsoft.com/en-us/nuget/consume-packages/configuring-nuget-behavior.

  -h|--help         Show help information

If -name param matches existing source, removes the source.


### Usage: dotnet nuget enable source [options]

Options:
  -n|--name <name>  Name of the source.

  -c|--configfile   The NuGet configuration file. If specified, only the settings from this file will be used. If not specified, the hierarchy of configuration files from the current directory will be used. To learn more about NuGet configuration go to https://docs.microsoft.com/en-us/nuget/consume-packages/configuring-nuget-behavior.

  -h|--help         Show help information

If -name param matches existing source, enables the source.


### Usage: dotnet nuget disable source [options]

Options:
  -n|--name <name>  Name of the source.

  -c|--configfile   The NuGet configuration file. If specified, only the settings from this file will be used. If not specified, the hierarchy of configuration files from the current directory will be used. To learn more about NuGet configuration go to https://docs.microsoft.com/en-us/nuget/consume-packages/configuring-nuget-behavior.

  -h|--help         Show help information

If -name param matches existing source, disables the source.

Consider Post-MVP Improvement:
* [#8668](https://github.com/NuGet/Home/issues/8668)
* do 2 fixes, make case match...but support non-matching cases.
* should we persist as lower case?
* For MVP, we've made a change to persist the disabled source name as the same case as the source. can consider 2nd fix to support the non-matching case, post-MVP.