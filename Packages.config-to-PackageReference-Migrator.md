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
Every Visual Studio customer not using packages.config based projects. 

## Evidence
A lot of customer feedback on packages.config, issues with dependency management, performance issues that we think moving to PackageReference solves. 

## Solution

### Non Goals
While a solution level upgrade experience is desirable, we will start with project level upgrade first.