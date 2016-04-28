## Feedback

If you have any ideas or concerns with this design, please comment on the following GitHub issue:
https://github.com/NuGet/Home/issues/2476

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

My proposal is to introduce a new type of child node to the `<metadata>` element which allows tooling to act differently for .NET CLI tools. The new child element is the `<packageType>` element. There can be zero or more `<packageType>` elements that indicate types of this package. Each `<packageType>` element must have a `type` attribute. Supported values of the `type` attribute are:

- `dotnet-cli-tool` - indicates that this package is a .NET CLI tool and should be installed to the `"tools"` node of the consuming project.json file.
- `dependency` - indicates that this package is a dependency. All packages without any explicit `<packageType>` are assumed to be of the `dependency` type. This includes are packages predating this specification.

### dotnet-cli-tool .nuspec

<pre>
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;package xmlns="http://schemas.microsoft.com/packaging/2012/06/nuspec.xsd"&gt;
  &lt;metadata&gt;
    ...
    <b>&lt;packageType type="dotnet-cli-tool" /&gt;</b>
  &lt;/metadata&gt;
&lt;/package&gt;
</pre>

### dependency .nuspec

<pre>
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;package xmlns="http://schemas.microsoft.com/packaging/2012/06/nuspec.xsd"&gt;
  &lt;metadata&gt;
    ...
    <b>&lt;packageType type="dependency" /&gt;</b>
  &lt;/metadata&gt;
&lt;/package&gt;
</pre>

## Tool Creation

To create a package that can operate as a .NET CLI tool, there is one option (in addition to manually editing the input .nuspec file to a `<packageType>` node. The developer adds a `"packageType"` key to the existing `"packOptions"` node of the project.json of the .NET tool that is being created.

For example, this could be the project.json of the `dotnet-hello` tool described above.

```json
{
  "version": "2.0.0-*",
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
  "packOptions": {
    "packageType": "dotnet-cli-tool"
  }
}
```

The format of a package type string is exactly like a package ID. That is, a package type is a case-insensitive string matching the regular expression `^\w+([_.-]\w+)*$` having at least one character and at most 100 characters. 

Any string following these rules can be specified as the `packageType` in a project.json. The pack command will simply copy this string to the output .nuspec `<packageType>` node. If no value is specified, the pack command will default to a package type of `<packageType type="dependency" />`.

If more than one value is supplied (e.g. via a JSON array), the pack command fails.

## Installation

The behavior of NuGet's install command will be modified so that when a `dotnet-cli-tool` package is being installed to a .NET CLI project.json file, the package will be added to the `"tools"` node instead of the `"dependencies"` node. Installation of `dotnet-cli-tool` packages to non-.NET CLI project.json or to packages.config behaves exactly as a dependency.

If a package has no explicit package type, the package is assumed to be a dependency. If a package has an unrecognized (not `dotnet-cli-tool` or `dependency`) type or more than one type, the installation fails.

## Restoration

A package's `<packageType>` does not effect NuGet's restore operation. For example, if a project has a transitive dependency that is a `dotnet-cli-tool`.

## Existing packages

Treatment of existing packages will not change. Since the notion of type is not a single-valued or required concept, existing packages do not need any type to be explicitly added or even inferred.