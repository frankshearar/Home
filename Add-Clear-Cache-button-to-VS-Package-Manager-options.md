## Problem
Allow users to clear NuGet cache from within Visual Studio

## Who is the customer?
Package authors, consumers.

## Evidence
Button existed on previous version of NuGet.

## Solution
Add a button to the VS Package Manager options which calls into the LocalsCommandRunner.cs from NuGet.Core.CommandLine.Commands. 
The call is similar as `nuget locals -clear all` from nuget CLI or `dotnet nuget locals --clear all` from dotnet CLI.

The result is displayed on a popup/message box.