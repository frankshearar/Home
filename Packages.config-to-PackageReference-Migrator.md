# Packages.config to PackageReference Migrator : Helping users to migrate to transitive world of package reference

## Problem
Currently, a large number of our customers are using packages.config. 
Packages.config has several drawbacks, namely
* Solution level packages folder
* Flat dependency lists. Dependencies that are not directly referenced end up in packages.config making dependency management hard.
* Packages.config projects enable modification of Project files (reference data is duplicated) and makes clean uninstall almost impossible
* Finally, there is large security risk in enabling the execution of custom powershell scripts (Install.ps1) during packages install and unknown set of changes happening to your project.

Currently, migrating from packages.config to PackageReference is a hard problem and a manual process that could easily result in projects not building/running correctly. We want to make it easy for customers to migrate to PackageReference. 
The proposed solution is to provide a Visual Studio command to migrate these projects. 

## Open Issues
TODO

## Who is the customer?
Every Visual Studio customer not using packages.config based projects. We want to get users to move away from the dependency management hell some of them are finding themselves in. Future investments will be primarily on top of new standards like PackageReference and we want to bring all our customers in for a ride. Currently, users have to use either read blogs or this [doc] (https://blog.nuget.org/20170316/NuGet-now-fully-integrated-into-MSBuild.html) to do this themselves. Package authors and consumers both will be hugely benefit from this change.

## Evidence
A lot of customer feedback on packages.config, issues with dependency management, performance issues that can essentially be solved using PackageReference.

## Solution
At a uber level, this feature will provide upgrade command to customers inside visual studio to upgrade managing individual project's nuget dependencies from packages.config to PackageReference. 

### Non Goals
While a solution level upgrade experience is desirable, we will start with project level upgrade first.

### Upgrade Flow
* As a first step, this feature will only be available in Visual studio at two surface areas 1) at NuGet manager UI at project level 2) right-click menu option on project. User invoke upgrade by either mean
   * A dialog comes up explaining to the user the exact changes that will happen to the project. And also gives two options to upgrade:
      * Collapse dependencies (recommended), this will be default selected until user changes it which will be persisted. This option allows to only install top level packages and exclude their dependencies which will anyway be available as transitive dependencies.
      * Flatten dependencies, this will also to install all dependencies from packages.config as top level dependencies in PackageReference.
* Once user clicks OK button, we bring standard VS blocking UI progress bar (like we had in previous solution restore).
* It will then backup current project file as well as packages.config file to different location so that if anything goes wrong, user can always go back to the previous state.
* It will then start uninstalling all the packages from packages.config.
   * If any of the uninstall fails, then it will rollback all the changes, report error on output console with relevant details, and stop upgrade.
* Then, it will start installing packages as PackageReference. It will also take care of P2P references evaluation to bring transitive dependencies as well as parent projects reevaluation.
   * If, installation fails, then
      * it will report detailed error message on output console
      * backup path location to retrieve previous project file as well as packages.config
      * and a link to simple instructions to go back to the previous state
* Finally, after completing a successful upgrade, it will show an upgrade report summary, which lists:
   * packages installed as PackageReference
   * packages skipped as top level dependencies
   * issues found with any of these packages while working with PackageReference
   * path location for backup files
   * link to instructions to revert these changes

### Mockups
* Options to trigger this upgrade
 1) right click menu option at project
![](https://github.com/NuGet/Home/blob/dev/resources/MigratorToolSupport/right%20click%20project%20menu.png)

 2) NuGet manager UI option at project level
![](https://github.com/NuGet/Home/blob/dev/resources/MigratorToolSupport/manager%20UI%20project%20upgrade%20option.png)

* Upgrade preview window
![](https://github.com/NuGet/Home/blob/dev/resources/MigratorToolSupport/upgrade%20preview%20window.png)

* Upgrade summary
![](https://github.com/NuGet/Home/blob/dev/resources/MigratorToolSupport/upgrade%20summary.png)