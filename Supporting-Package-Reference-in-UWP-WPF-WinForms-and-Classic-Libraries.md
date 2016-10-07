## Issue
Spec for the work is available here: - https://github.com/NuGet/Home/issues/3561

## Problem
Currently there are multiple ways (packages.config, project.json and PackageReference in csproj) of managing dependencies and we want to standardize on one global way of managing dependencies. Its hard on customers to read and understand about multiple ways of managing dependencies. 

## Who is the customer?
All .NET Customers who want to move towards managing dependencies via the model built for project.json.

## Solution

There are 2 key scenarios at play here.

* Customers who want to move from package.config to PackageReference
* Customers who want to move from project.json to PackageReference

### Supported VS Versions

One thing to note is that PackageReference support is only available in VS "15" RTM+ versions of Visual Studio and the corresponding versions of NuGet.exe. Customers upgrading to PackageReference will be unable to roundtrip their projects with previous versions of Visual Studio including VS 2015

### PackageReference behavior in VS 2015
If projects with PackageReference is opened in the downlevel version of NuGet package manager that will be shipped with VS 2017. We will show a error in the errorlist when a restore is invoked on a project that contains a package reference or any operations through PMC or the Package Manager UI is tried on that project.

**Message:** Managing NuGet packages through PackageReferences is only supported in Visual Studio 2017 and above. Revert to project.json or packages.config to manage NuGet packages for {ProjectName}. To learn more, refer to following link https://aka.ms/packagereferencesupport



