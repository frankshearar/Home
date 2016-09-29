

## Issue
Link to the GitHub issue that tracks the work and discussion.

**ToolsReference:** https://github.com/NuGet/Home/issues/3505
**PackageReference UI:** https://github.com/NuGet/Home/issues/3457


## Problem
With the advent of .NET Core, we need to support addition of ToolsReferences from within the NuGet package manager and PMC. In addition, we also need 

## Who is the customer?
.NET Core customers for ToolsReferences and all .NET customers in VS for PackageReference

## Solution

###ToolsReferences

Packages with PackageType set to DotnetCliTool or set to DotnetCliTool and Dependency are treated as Tools Packages. When these packages are installed, they are added as a DotnetCliToolReference or a DotnetCliToolReference and PackageReference. The following are the key use-cases and default we should use in our UI.

* Packages with Package type set to DotnetCliTool can never be installed **only** as a PackageReference. Conversely, packages with no package type or package type set to only Dependency cannot be installed as a DotnetCliToolReference.
* If the package type is set to DotnetCliTool only, then by default we install it only as a DotnetCliToolReference but we will provide an option in the UI that will enable folks to install it as a PackageReference as well. 
* If the package type is set to DotnetCliTool and Dependency , then by default we install it as a DotnetCliToolReference and a PackageReference but users can choose to turn of installation as a PackageReference.

####UI
Users can change the defaults by expanding the options expander in the package details page in the UI. We should use a combo-box prepopulted with the following options and set the default based on the use cases listed above.

**Label**: Install as:

**Options**

* Tool
* Dependency and Tool

### PMC Commands

We will need to enable these options to be passed to the Install/Update commands in the PMC. Currently this work is not planned for the RC time-frame.

**Argument** - InstallAs

**Possible Values** - Tool, Dependency (comma separated multiple values)

### Multi-TFM Support





