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

My proposal is to introduce a new child node to the `<metadata>` element which allows tooling to act differently for .NET CLI tools. The new child element is the `<types>` element. This `<types>` element can contain zero or more child elements that indicate types of this package.

Supported children of the `<types>` element are:

- `<dotnet-cli-tool>` - indicates that this package is a .NET CLI tool and should be installed to the `"tools"` node of the consuming project.json file.

<pre>
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;package xmlns="http://schemas.microsoft.com/packaging/2012/06/nuspec.xsd"&gt;
  &lt;metadata&gt;
    ...
    <b>&lt;types&gt;
      &lt;dotnet-cli-tool /&gt;
    &lt;/types&gt;</b>
  &lt;/metadata&gt;
&lt;/package&gt;
</pre>

## Additional concerns

### Multiple types

Although there is currently no need for multiple package types, the intent of this design is to allow for a package to have multiple types. For example, if a package can be treated both as a .NET CLI tool and, say, a "Win32 tool", we could mark the package like this:

<pre>
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;package xmlns="http://schemas.microsoft.com/packaging/2012/06/nuspec.xsd"&gt;
  &lt;metadata&gt;
    ...
    <b>&lt;types&gt;
      &lt;dotnet-cli-tool /&gt;
      &lt;win32-tool /&gt;
    &lt;/types&gt;</b>
  &lt;/metadata&gt;
&lt;/package&gt;
</pre>

### Existing packages

Treatment of existing packages will not change. Since the notion of type is not a single-valued or required concept, existing packages need to type to be inferred.

### Searchability and indexing

If we wanted to provide a feature on NuGet.org to search for any kind of tool package, we could build knowledge into the search index to look for the `<dotnet-cli-tool>`. However, if someone introduces a new kind of tool package that is not meant for the .NET CLI, e.g. `<win32-tool>`, we would have to add an additional rule to the server asserting that <win32-tool>` is also a tool should be included in the generic tool search.

To address this problem, we could also mark this tool with an additional type:

<pre>
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;package xmlns="http://schemas.microsoft.com/packaging/2012/06/nuspec.xsd"&gt;
  &lt;metadata&gt;
    ...
    <b>&lt;types&gt;
      &lt;dotnet-cli-tool /&gt;
      &lt;tool /&gt;
    &lt;/types&gt;</b>
  &lt;/metadata&gt;
&lt;/package&gt;
</pre>

This would allow any indexing procedure to write more generic code searching for the `<tool>` element. For this to work, however, an additional burden would be placed on the tool producer (either explicit user intervention or `pack` command code) to add this additional element every time a `<dotnet-cli-tool>` is produced.