# Converting Extension SDKs into NuGet Packages

## Issue
Feedback on the spec can be given on the following issue - https://github.com/NuGet/Home/issues/3509

## Problem
Currently ISVs and First Party Library (Framework packages) end up shipping Extension SDKs to support a variety of UWP scenarios. The reasons are both historical and in some cases NuGet does not have the complete feature set that Extension SDKs have.

Extension SDKs are inherently not very friendly to developers since they require to be installed on build/dev machines in-order for solutions to compile and does not support the seamless package management workflow that is built into NuGet.

## Who is the customer?
First Party Framework SDK authors and ISVs.

## Evidence
We have 3 primary customers for this. .NET Native, Store Services and Controls @ Speed from the XAML Framework team.

## Solution
There are 3 major features that we need to support to in-order to provide a seamless transition for Extension SDK authors to use NuGet.

### Appx Packages
Framework SDKs require registration of Appx packages during deployment. This can be accomplished using currently available NuGet semantics.

Appx files need to be placed in the following relative location of the NuGet package

    \tool\*

Specifically they should be tied to the architecture of the Appx package. E.g

    \tool\x86\MyTestpackage.appx
    \tool\x64\MyTestpackage.appx
    \tool\ARM\MyTestpackage.appx
    \tool\ARM64\MyTestpackage.appx

The UWP tools are able to parse these paths from the lock file and correspondingly register them during deployment. This is a feature that is unique to first party packages since only they can be store serviced.




