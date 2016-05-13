# NuGet Upgrader: Helping users move to the awesomeness of NuGet 3.0 Standard

## Problem
A large number of customers who are not UWP, .NET Core, and ASP.NET Core are currently using packages.config. Packages.config has several drawbacks, namely
* Flat dependency lists. Dependencies that are not directly referenced end up in packages.config making dependency management hard.
* Packages.config projects enable modification of Project files (reference data is duplicated) and makes clean uninstall almost impossible
* Finally, there is large security risk in enabling the execution of custom powershell scripts (Install.ps1) during packages install and unknown set of changes happening to your project.

Currently, converting from packages.config to nuget/project.json is a hard problem and a super manual process that could easily result in projects not building or running correctly. We want to help make it super easy for customers to move to the new awesome world of project/nuget.json. Our solution provides an upgrade mechanism in Visual Studio to help people move to the new NuGet 3.0 standard. 

## Open Issues
However, there are some open issues here that need to be thought through:-
* We have not still decided on whether there will be a project/nuget.json or if all the dependencies will flow into the csproj.
* The timeline is a bit fuzzy at this point. Early adopters of this tool might have to do 2-3(max) upgrades over time to completely move over to the new standard.

## Who is the customer?
Every Visual Studio user not using NuGet Standard 3.0 is a potential customer. We want to get users to move away from the dependency management hell some of them are finding themselves in. Future investments will be primarily on top of NuGet Standard 3.0 and we want to bring all our customers in for a ride. Currently, users have to use either read blogs or this [doc] (https://github.com/NuGet/Home/wiki/Converting-a-csproj-from-package.config-to-project.json) to this themselves. Package authors and consumers both will be hugely benefit from this change.

## Evidence
-- Yishai's section

## Solution
