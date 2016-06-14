# NuGet Upgrader: Helping users move to the awesomeness of NuGet 3.0 Standard

## Problem
A large number of customers who are not UWP, .NET Core, and ASP.NET Core are currently using packages.config. Packages.config has several drawbacks, namely
* Solution level packages folder
* Flat dependency lists. Dependencies that are not directly referenced end up in packages.config making dependency management hard.
* Packages.config projects enable modification of Project files (reference data is duplicated) and makes clean uninstall almost impossible
* Finally, there is large security risk in enabling the execution of custom powershell scripts (Install.ps1) during packages install and unknown set of changes happening to your project.

Currently, converting from packages.config to nuget/project.json is a hard problem and a super manual process that could easily result in projects not building or running correctly. We want to help make it super easy for customers to move to the new awesome world of project/nuget.json. Our solution provides an upgrade mechanism in Visual Studio to help people move to the new NuGet 3.0 standard. 

## Open Issues
However, there are some open issues here that need to be thought through:-

* Transition to MSBuild projects: Dependencies in the neat future are going to be part of MSBuild projects. The tentative timeline for this is fall of this year. We need evaluate if there is additional burden for developers to move twice to the new format.

## Who is the customer?
Every Visual Studio user not using NuGet Standard 3.0 is a potential customer. We want to get users to move away from the dependency management hell some of them are finding themselves in. Future investments will be primarily on top of NuGet Standard 3.0 and we want to bring all our customers in for a ride. Currently, users have to use either read blogs or this [doc] (https://github.com/NuGet/Home/wiki/Converting-a-csproj-from-package.config-to-project.json) to this themselves. Package authors and consumers both will be hugely benefit from this change.

## Evidence
During a number of customer calls and during the User Study, we have seen that customers are running into issues that can be essentially solved with project.json on a day to day basis.

## Solution

### Non Goals
While a solution level upgrade experience is desirable, we will start with project level upgrade first.

### Experimental Features
This feature will need to evolve over a period of time so we will introduce a new Experimental features option that will be by default turned off. Have a single _Enable experimental features_ check-box in the General Page of NuGet Package Manager in Tools->Options.

**Text**: _Enable experimental features_

**Info Icon:** Not designed yet, we need to have a link that points to doc page that talks about experimental features

//TODO (Mockup)

### Supported Frameworks, Languages and Project Types

**Languages (If MSBuild based)**
* C#
* VB
* C++/CLI
* F#? - Need to discuss with ML?
* C++? - No

**Supported Frameworks**
* All Versions (E.g 1.1.1,  2.0, 3.X. 4.X.) + PCL Profiles

**Supported Projects**
* WPF
* WinForms
* Console App
* Class Libraries
* PCLs
* Windows Service
* VS Extensibility
* Test 
* Web (ASP.NET 4)
* Android
* iOS
* Office Projects
* VS Extensibility
* WCF
* Workflow
* Test 

**Blacklist(No new investments here)**
* Windows 8.1 Store and Phone 
* Silverlight
* Lightswitch

### Surfacing in Visual Studio
* Below Manage NuGet Packages in Project Context Menu and Top level menu.
* In the context menu for packages.config.
* In the Package Manager UI as a info bar that can be turned off.

**Command Text:** _Upgrade to NuGet 3.4 Standard_

**Info Bar Text:** _Upgrade to NuGet 3.4 Standard_

//TODO (Mockups)

### Upgrader flow
* User clicks on the Upgrade Command
   * A dialog comes up explaining to the user the exact changes that will happen to the project
      * Icon: Information
      * Text: <Detailed explanation of what this will mean>
      * Analysis Results
        * If there are any missing packages, we will not allow users to proceed (no OK button)
        * Show Results (issues that could happen if packages have content files, packages from install.ps1)
      * We need two options
        * Collapse dependencies (By Default)
        * Keep dependencies flat
      * Buttons: Cancel, OK
   * User clicks on Cancel - Dialog goes away
   * User clicks OK
      * We bring up the standard VS blocking UI progress bar (Like Restore). The following steps are documented in the dialog
	* Removal of packages.config <probably instantaneous>
        * Adding of project.json <probably instantaneous>
        * Adding top level packages
        * Restoring packages
   * Completion Dialog
      * Result: Upgrade Complete
      * **Text:** Please build and run your solution to verify that all packages are available
      * ** Text:** If you run into any issue and would like to revert, we have backed up changed files to _Location _. Follow the directions described in the following document to revert the changes.

![](https://github.com/NuGet/Home/blob/dev/resources/upgraderImage.png)