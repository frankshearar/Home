Status: **Incubation**

## Issue
The work for this feature and the discussion around the spec is tracked here - **packages.config (PC) to PackageReference (PR) Migrator [#5877](https://github.com/NuGet/Home/issues/5877)**

## Problem
More than 80% projects in Visual Studio use packages.config (PC). The PC format has several drawbacks such as:
* Package cache is duplicated across solution due to solution level package folder
* Due to a flat dependency structure, indirect dependencies end up cluttering the packages.config file which is not human readable.
* Allows modification of project files (reference data is duplicated) and makes clean uninstall almost impossible
* Allows execution of custom PowerShell scripts (Install.ps1) during package install which is a significant security risk

In addition to overcoming these drawbacks, PackageReference (PR) has several advantages as called out in this [blog post](https://blog.nuget.org/20170316/NuGet-now-fully-integrated-into-MSBuild.html).

Currently, users do not have an automated way to migrate projects from PC to PR. Manual migration is error prone and could result in broken projects.

## Who is the customer?
Every Visual Studio user that works on a PC based project and such projects account for more than 80% of all projects.

## Evidence
A lot of customer feedback on PC, issues with dependency management, performance issues that can essentially be solved using PackageReference.

## Solution
At a high level, this feature will provide gestures within Visual Studio that allow a user to migrate a packages.config project to PackageReference.

### Non-Goals
While a solution level migration experience may be desirable, we will start with the project level migrate experience first.

### Migrate Flow
* User invokes the migrator from the context menu for References in the solution explorer or the prompt in the NuGet package manager UI.

  ![](https://github.com/NuGet/Home/blob/dev/resources/MigratorToolSupport/PMUI%20with%20gold%20bar%20migrate.PNG)
  
* The migrator dialog is displayed.

  ![](https://github.com/NuGet/Home/blob/dev/resources/MigratorToolSupport/MainUpgraderUI%20v3.png)
  
* User reviews the packages that are classified as top-level and transitive, associated warnings, and clicks 'OK' to begin the migration. At this stage, the user has the option to force a package classified as transitive, to be treated as a top-level package by checking the top-level check box.
* The current project file and the packages.config file is backed up to the following path `<SolutionFolder>/Backup_{shortGuid}/<ProjectName>/`.
* The migrator will then start installing packages identified as a direct dependency as PackageReference. It will also take care of P2P references evaluation to bring transitive dependencies as well as parent projects reevaluation.
   * If, the installation fails, then
      * it will report detailed error message on output console
      * backup path location to retrieve previous project file as well as packages.config
      * and a link to instructions to go back to the previous state
* Finally, after completing a successful migration, it will show a migration report summary:

  ![](https://github.com/NuGet/Home/blob/dev/resources/MigratorToolSupport/report.PNG)
 
### Migration report
A migration report is generated and displayed after a migrate attempt.
The migration is successful if all the top-level packages were successfully installed as PackageReference.

### Top-level packages
A package that meets any of the following conditions will be treated as a top-level package:
* No other package expresses it as a dependency
* The package version brought in by restore (if treated as a transitive dependency) is different from the one referenced in packages.config
* The package contains assets that don't flow transitively by default in PR i.e. build, contentFiles, analyzers
* The package has the `developmentDependency` flag set to true

### Warnings and errors
* package contains a dll file at the root of the lib folder and not a TFM specific folder. For example - lib\foo.dll
* package contains install.ps1
* package contains xdt transforms
* package contains content but no contentFiles