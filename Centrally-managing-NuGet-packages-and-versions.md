## Issue
| Requirement | Issue |
|:---|:---|
| Developers would like to control the packages and their versions that are allowed to be used across the team/product | [6764](https://github.com/NuGet/Home/issues/6764) |
# Context
[PackageReference requirements summary](https://github.com/NuGet/Home/wiki/PackageReference-enhancements) | Epic issue [#6763](https://github.com/NuGet/Home/issues/6763)

## Who is the customer?
Enterprise customers with huge code-base spanning 100s of projects. 

## Problem
#### Developers would like to specify control the packages and their versions that are allowed to be used in their projects or solutions across the team/product

| PH# | Problem Hypothesis |
|:--- |:---------------|
| PRS7 | Developers cannot define an allowed list of packages that can be used in an application development across projects/solutions/repos |
| PRS8 | Developers cannot restrict to use the same version of a given package across projects/solutions/repos |
| PRS9 | Developers find it difficult to use a predetermined allowed version range of given package across projects/solutions/repos |

With huge code bases, complex project structures with large set of packages to deal with, it becomes really hard for developers to keep consistency on the package and versions used across the projects/solutions. Developers need a way to control the packages they would like to use downstream from a repository level or a solution level. They would like to ensure that only specific versions can be used throughout their code base.

## Evidence
_What is the evidence that behaves us to act?_
_Evidence can be impassioned tweets or mails, mile long conversations in issues, rants on blogs, sweet sweet data from telemetry!!! If you can show pain, you can rally the troops._

## Solution
_Detailed explanation of the solution. The more pictures/code snippets based on the feature the merrier. Pictures keep folks awake when reading specs._