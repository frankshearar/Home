# Title
Adding pack as a msbuild target for csproj.

## Problem
As part of an effort to move restore, build, package and publish to a unified msbuild pipeline in cross platform environments, we need to have a fully .NET Core implementation of pack. dotnet pack will need to be replaced to call into the new pack target in msbuild that can run cross platform and support cross targeting scenarios. As a result, project.json will go away and all metadata and package references from nuspec and project.json will move into csproj.

## Who is the customer?
_Who is the customer that is running into the problem. Which customers would dance for joy and donate to save the space unicorns foundation on getting this feature. Customers here could be individuals, nuget customer segments (package authors, consumers), enterprises, partners within Microsoft, external partners etc..._

## Evidence
_What is the evidence that behaves us to act?_
_Evidence can be impassioned tweets or mails, mile long conversations in issues, rants on blogs, sweet sweet data from telemetry!!! If you can show pain, you can rally the troops._

## Solution
_Detailed explanation of the solution. The more pictures/code snippets based on the feature the merrier. Pictures keep folks awake when reading specs._