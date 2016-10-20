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

**Message:** Managing NuGet packages through PackageReferences is only supported in Visual Studio 2017 and above. Revert to project.json or packages.config to manage NuGet packages for {ProjectName} in this version of Visual Studio. To learn more, refer to following link https://aka.ms/packagereferencesupport.


### PackageReference behavior in VS 2017+
At RTM, PackageReference will only be added if project.json and packages.config are missing in a project and/or a PackageReference is detected in the project, If a PackageReference is detected, future references will be added as PackageReference and all NuGet actions will be performed on this basis. 

The user will have to explicitly migrate existing dependencies to PackageReference. Automatic migration will not be available at RTM. In other classic desktop projects or PCLs when a user tries to add the first NuGet package, will bring up a dialog that gives them a choice a using packages.config or PackageReference to manage their dependencies.

********************************* TBD***************************************************
**Title:** Choose NuGet Package Management Format

**Message:** In Visual Studio 2017 and above, NuGet Packages can be managed using PackageReferences in your project file. To learn more about PackageReferences, refer to the following link https://aka.ms/packagereferencesupport

** Checkbox**: Dont show this message again. This will set your default option as packages.config. You can change your default in NuGet settings in Tools and Option.

**Buttons**: Choose Packages.config, Choose PackageReference

In addition to this we will have a new option in NuGet Options in Visual Studio that will specify the default behavior that is set when users either 

The default behavior is to keep using packages.config for classic projects. You can use the option to change it to Package References.










