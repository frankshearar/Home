## Problem
Allow users to clear NuGet cache from within Visual Studio

## Who is the customer?
Package authors, consumers.

## Evidence
Button existed on previous version of NuGet. This work is related to [enabling locals](https://github.com/NuGet/Home/wiki/Support-locals-command-in-dotnet-cli) command in dotnet cli.

## Solution
Add a button to the VS Package Manager options which calls into the LocalsCommandRunner.cs from NuGet.Core.CommandLine.Commands. 
The call is similar as `nuget locals -clear all` from nuget CLI or `dotnet nuget locals --clear all` from dotnet CLI.

<strike>The result is displayed on a popup/message box.</strike>

The result will be displayed in a label below the Clear cache button along with the time stamp of completion of the operation. The error messages also contain a link to a help page. The screenshots below have been updated.

## Screenshots

![Clear Cache Button](https://cloud.githubusercontent.com/assets/10507120/17826473/f9382e90-6625-11e6-9b3d-a7bc19b779f6.png)

![Waiting for Clear cache operation](https://cloud.githubusercontent.com/assets/10507120/17826480/0471d0cc-6626-11e6-9939-dc8056e9d1da.png)

![Success Message](https://cloud.githubusercontent.com/assets/10507120/17826479/0470cd76-6626-11e6-95f6-4fd1f935773d.png)

![Failure Message](https://cloud.githubusercontent.com/assets/10507120/17826485/1176b8c8-6626-11e6-87e4-82beebf69dec.png)
***
