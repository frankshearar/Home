## Problem
Allow users to clear NuGet cache from within Visual Studio

## Who is the customer?
Package authors, consumers.

## Evidence
Button existed on previous version of NuGet.

## Solution
Add a button to the VS Package Manager options which calls into the LocalsCommandRunner.cs from NuGet.Core.CommandLine.Commands. 
The call is similar as `nuget locals -clear all` from nuget CLI or `dotnet nuget locals --clear all` from dotnet CLI.

<strike>The result is displayed on a popup/message box.</strike>

The result will be displayed in a label below the Clear cache button along with the time stamp of completion of the operation. The error messages also contain a link to a help page. The screenshots below have been updated.

## Screenshots

![Clear Cache Button](https://cloud.githubusercontent.com/assets/10507120/17796340/96a187d8-6574-11e6-808b-077eab738780.png)

![Waiting for Clear cache operation](https://cloud.githubusercontent.com/assets/10507120/17796346/9c0885f0-6574-11e6-8843-3259e40d0180.png)

![Success Message](https://cloud.githubusercontent.com/assets/10507120/17796349/9f3c3ed8-6574-11e6-9fc4-0629f51150f3.png)

![Failure Message](https://cloud.githubusercontent.com/assets/10507120/17796352/a3d44c24-6574-11e6-8b25-7b698d86b71a.png)
***
