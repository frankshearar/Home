# IN PROGRESS

## Goal

The goal of this document is to propose a way for .NET CLI tool packages to be marked in such a way that NuGet tooling can install these packages to the `"tools"` node of a project.json. Without this proposal, packages (tool or otherwise) will be installed to the `"dependencies"` node, which is the current functionality for all existing packages.

Here's an example of a project.json file with a `"tools"` dependency.

```json
{
  "version": "1.0.0-*",
  "compilationOptions": {
    "emitEntryPoint": true
  },
  "dependencies": {
    "NETStandard.Library": "1.5.0-rc2-24008"
  },
  "frameworks": {
    "netstandardapp1.5": {
      "imports": "dnxcore50"
    }
  },
  "tools": {
    "dotnet-hello": "1.0.0"
  }
}
```

## History

Today, there is nofirst class notion of package type. However, packages can be used for different things based on their content. The most common use for a package is to be an assembly used at run-time. However, other packages drop static content in your project, provide executable tools, provide MSBuild targets, or have templated code files. Packages can also contain no content at all and simply pull in other packages (i.e. metapackages). Aside from inspecting the folder structure of the package, there is no way to differentiate between packages.

## Today's .nuspec

Every package contains a .nuspec file which provides metadata and dependency information about the package. This is the natural place for any notion of package type. Here is an example .nuspec file for the tool package (used by .NET CLI tests) `dotnet-test`.

```xml
<?xml version="1.0" encoding="utf-8"?>
<package xmlns="http://schemas.microsoft.com/packaging/2012/06/nuspec.xsd">
  <metadata>
    <id>dotnet-hello</id>
    <version>2.0.0</version>
    <authors>dotnet-hello</authors>
    <owners>dotnet-hello</owners>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>dotnet-hello</description>
    <tags></tags>
    <dependencies>
      <group targetFramework=".NETStandardApp1.5">
        <dependency id="NETStandard.Library" version="[1.5.0-rc2-24008, )" />
      </group>
    </dependencies>
  </metadata>
</package>
```

## Proposal

My proposal is to introduce a new element under  `<metadata>` which allows tooling to act differently for .NET CLI tools. The element is `<types>` and will start with one supported child element:

  - `<dotnet-cli-tool />` - indicates this tool is a .NET CLI tool and should be installed to the `<tools>` node of the project.json.

In general, as with the rest of the .nuspec, children of the `<types>` element that are not recognized should be ignored. Multiple types are supported.

```xml
<?xml version="1.0" encoding="utf-8"?>
<package xmlns="http://schemas.microsoft.com/packaging/2012/06/nuspec.xsd">
  <metadata>
    ...
    <types>
      <dotnet-cli-tool />
    </types>
  </metadata>
</package>
```

## Thoughts