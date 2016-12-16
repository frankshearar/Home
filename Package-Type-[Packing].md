## Feedback

If you have any ideas or concerns with this design, please comment on the following GitHub issue:
https://github.com/NuGet/Home/issues/2476

## Revisions

- **2016-04-28** - Initial accepted design.
- **2016-05-05** - Change style of package type names and add notion of version.
- **2016-05-06** - Add extended example of a custom package type.
- **2016-06-13** - Added parent `<packageTypes>` element, since the .nuspec XSD uses `<xsd:all>`.
- **2016-06-13** - Update the .nuspec XML namespace to match the new schema.
- **2016-07-07** - Clarify installation of unsupported package type to non-.NET Core projects.
- **2016-07-14** - Do not use a new XML namespace in the .nuspec when package types are used.
- **2016-12-16** - Package type name uses the "name" attribute instead of "type".

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

Today, there is no first class notion of package type. However, packages can be used for different things based on their content. The most common use for a package is to be an assembly used at run-time. However, other packages drop static content in your project, provide executable tools, provide MSBuild targets, or have templated code files. Packages can also contain no content at all and simply pull in other packages (i.e. metapackages). Aside from inspecting the folder structure of the package, there is no way to differentiate between packages.

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

My proposal is to introduce a new type of child node to the `<metadata>` element which allows tooling to act differently for .NET CLI tools. The new child element is a `<packageTypes>` element with one or more `<packageType>` children. There can be zero or more `<packageType>` elements that indicate types of this package. Each `<packageType>` element must have a `type` attribute. Supported values of the `type` attribute are:

- `DotnetCliTool` - indicates that this package is a .NET CLI tool and should be installed to the `"tools"` node of the consuming project.json file.
- `Dependency` - indicates that this package is a dependency. All packages without any explicit `<packageType>` are assumed to be of the `Dependency` type. This includes are packages predating this specification.

In both cases, the version should not be specified. The version defaults to `0.0` when not specified.

### DotnetCliTool .nuspec

<pre>
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;package xmlns="http://schemas.microsoft.com/packaging/2012/06/nuspec.xsd"&gt;
  &lt;metadata&gt;
    ...
    <b>&lt;packageTypes&gt;
      &lt;packageType name="DotnetCliTool" /&gt;
    &lt;/packageTypes&gt;</b>
  &lt;/metadata&gt;
&lt;/package&gt;
</pre>

### Dependency .nuspec

<pre>
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;package xmlns="http://schemas.microsoft.com/packaging/2012/06/nuspec.xsd"&gt;
  &lt;metadata&gt;
    ...
    <b>&lt;packageTypes&gt;
      &lt;packageType name="Dependency" /&gt;
    &lt;/packageTypes&gt;</b>
  &lt;/metadata&gt;
&lt;/package&gt;
</pre>

## Tool Creation

To create a package that can operate as a .NET CLI tool, there is one option (in addition to manually editing the input .nuspec file to add a `<packagesTypes>` and `<packageType>` node. The developer adds a `"packageType"` key to the existing `"packOptions"` node of the project.json of the .NET tool that is being created.

For example, this could be the project.json of the `dotnet-hello` tool described above.

```json
{
  "version": "2.0.0",
  "buildOptions": {
    "emitEntryPoint": true
  },
  "dependencies": {
    "Microsoft.NETCore.App": "1.0.0"
  },
  "frameworks": {
    "netcoreapp1.0": {}
  },
  "packOptions": {
    "packageType": "DotnetCliTool"
  }
}
```

The format of a package type string is exactly like a package ID. That is, a package type is a case-insensitive string matching the regular expression `^\w+([_.-]\w+)*$` having at least one character and at most 100 characters. 

Any string following these rules can be specified as the `packageType` in a project.json. The pack command will simply copy this string to the output .nuspec `<packageType>` node. If no value is specified, the pack command will default to a package type of `<packageType name="Dependency" />`.

If more than one value is supplied (e.g. via a JSON array), the pack command fails.

## Installation

The behavior of NuGet's install command will be modified so that when a `DotnetCliTool` package is being installed to a .NET CLI project.json file, the package will be added to the `"tools"` node instead of the `"dependencies"` node. 

If a package has no explicit package type, the package is assumed to be a dependency.

If a package has an unrecognized (not `DotnetCliTool` or `Dependency`) type, the installation fails.

Installation of the `DotnetCliTool` package type is only supported on .NET Core project.json projects. Installation of `DotnetCliTool` packages to non-.NET CLI project.json or to packages.config fails with a message explaining that the package type is unsupported.

If there is more than one package type, installation fails.

## Restoration

A package's `<packageType>` does not affect NuGet's restore operation. For example, if a project has a transitive dependency that happens to be a `DotnetCliTool`, this package is treated as a normal dependency.

For a `DotnetCliTool` to be treated as a tool and to be invokable with, say, `dotnet hello`, it must be mentioned explicitly in the consuming project's `"tools"` node. Conversely, a package in the `"tools"` node cannot be used as a dependency. 

## Existing packages

Treatment of existing packages will not change. Since the notion of type is not a single-valued or required concept, existing packages do not need any type to be explicitly added or even inferred.

## Other Package Types

As mentioned in the [Tool Creation](#tool-creation) section, the `"packageType"` property under `"packOptions"` in a project.json can be set to any string adhering to the aforementioned format rules.

Do note that the NuGet client (NuGet in Visual Studio) will reject installation of unrecognized package types.

For example, suppose a package creator was using NuGet packages to deliver a Windows desktop executable under the `tools` .nupkg directory. The package creator and consumer would then come to some agreement on a package type string. In this example, the package type string could be `Win32Tool`.

The package creator would craft a package with the following .nuspec:

<pre>
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;package xmlns="http://schemas.microsoft.com/packaging/2012/06/nuspec.xsd"&gt;
  &lt;metadata&gt;
    ...
    <b>&lt;packageTypes&gt;
      &lt;packageType name="Win32Tool" /&gt;
    &lt;/packageTypes&gt;</b>
  &lt;/metadata&gt;
&lt;/package&gt;
</pre>

This .nupkg could be created by manually editing the .nuspec file provided to the pack command. Alternatively, the package type could be specified in the project.json file.

<pre>
{
  ...
  "packOptions": {
    <b>"packageType": "Win32Tool"</b>
  }
}
</pre>

Finally, the package consumer would implement a client that downloads and extracts packages that have this package type (perhaps using NuGet client code to restore the package). A well behaved client should only interact with a specific list of package types. Package type is the first class way of knowing that a package was intended for scenarios supported by the client.

Other experiences can be improved by observing the package type. For example, package discovery (e.g. a search UI on a website) filter based on this package type.