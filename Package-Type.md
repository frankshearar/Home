# IN PROGRESS

## Goal

The goal of this document is to propose means that .NET CLI tool packages can be marked in such a way that NuGet tooling can install these packages to the `"tools"` node of a project.json. Without this proposed annotation, packages should be installed to the `"dependencies"` node, which is the current functionality for all existing packages.

## History

Historically, there has not been any first class notion of package type. Packages can be used for different things based on their content. The most common use for a package is to be an assembly used at run-time or at compile-time. However, other packages drop static content in your project or provide executable tools. Packages can also contain no content at all and simply pull in other packages (i.e. metapackages). Aside from inspecting the folder structure of the package, there is no way to differentiate between packages.
