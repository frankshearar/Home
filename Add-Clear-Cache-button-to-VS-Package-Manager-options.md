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

## Screenshots

![Clear Cache Button](https://cloud.githubusercontent.com/assets/10507120/17528699/4b0ebe04-5e25-11e6-8af0-feb8e1ff674e.png)

![Result Message](https://cloud.githubusercontent.com/assets/10507120/17746904/34d983f6-6467-11e6-8092-682dae58eea9.png)