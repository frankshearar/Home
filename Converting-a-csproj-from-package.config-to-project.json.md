### What is out there right now
Today (Dec 2015) there are a few officially supported project types using project.json to describe package dependencies officially.

These project types are UWP projects, Portable class libraries and ASP.NET 5 projects (class libraries, web applications and console applications).

These two can be divided into two:
1. Projects that rely on project.json as the main way to describe the build/deployment and packaging story
2. Projects that rely on project.json as the mechanism to define package dependencies (and perhaps later packaging), but rely on a csproj (or vbproj/fsproj) to define the rest of the build requirements.

### What is this document about
Other project types that do not yet officially support project.json can still benefit from the advantages project.json  brings. This is about how to get from a project with a packages.config file to one with project.json to enjoy:

1. No changes to csproj files (preventing merge conflicts)
2. No need to run nuget update (by using * notations in dependencies)
3. Transitive restore (allowing defining just top level dependencies)
4. P2P dependency flow (allowing for a top level app to always pick up dependencies from class libraries it depends on, without explicitly installing them).

### Getting started
Imagine a simple class library that installed Newtonsoft.Json package.

Here is the packages.config
```xml
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="Newtonsoft.Json" version="7.0.1" targetFramework="net45" />
</packages>
```

Adding a reference to package.config means that this package added a reference to newtonsoft.json.dll in the csproj file.
Note that it ands a hint path into the net45 folder of the package.

The side effects are
1. You can just edit packages.config
2. You can re-target the application without reinstalling the package.

```xml
  <ItemGroup>
    <Reference Include="Newtonsoft.Json, Version=7.0.0.0, Culture=neutral, PublicKeyToken=30ad4fe6b2a6aeed, processorArchitecture=MSIL">
      <HintPath>..\packages\Newtonsoft.Json.7.0.1\lib\net45\Newtonsoft.Json.dll</HintPath>
      <Private>True</Private>
    </Reference>
    <Reference Include="System" />
    <Reference Include="System.Core" />
    <Reference Include="System.Xml.Linq" />
    <Reference Include="System.Data.DataSetExtensions" />
    <Reference Include="Microsoft.CSharp" />
    <Reference Include="System.Data" />
    <Reference Include="System.Net.Http" />
    <Reference Include="System.Xml" />
  </ItemGroup>
  <ItemGroup>
    <Compile Include="Program.cs" />
    <Compile Include="Properties\AssemblyInfo.cs" />
  </ItemGroup>
```

#### Making the change
* Make a copy of packages.config and the csproj file and better yet, checkin/commit the project, so you can roll back or compare if things go wrong.

* Uninstall the package, you can use the UI, or the powershell console, if you know what the package did to the project, you can also hand edit the changes (2c).

UI: 

PowerShell console: uninstall-package newtonsoft.json -Force 

Manually: remove the lines:

```xml
    <Reference Include="Newtonsoft.Json, Version=7.0.0.0, Culture=neutral, PublicKeyToken=30ad4fe6b2a6aeed, processorArchitecture=MSIL">
          <HintPath>..\packages\Newtonsoft.Json.7.0.1\lib\net45\Newtonsoft.Json.dll</HintPath>
          <Private>True</Private>
        </Reference>
```
* Create the following empty file and name it project.json

```json
    {
        "dependencies": {
        "Newtonsoft.Json": "7.0.1"
        },
        "frameworks": {
            "net45": { }
        },
        "runtimes": {
            "win": { }
        }
    }
```
* Reload the solution (there is currently a bug where switching from packages.config to project.json requires a reload of the project).

* Build or restore packages (Right Click on the Solution level).

####Your project now uses project.json

```xml
    <ItemGroup>
        <Reference Include="System" />
        <Reference Include="System.Core" />
        <Reference Include="System.Xml.Linq" />
        <Reference Include="System.Data.DataSetExtensions" />
        <Reference Include="Microsoft.CSharp" />
        <Reference Include="System.Data" />
        <Reference Include="System.Net.Http" />
        <Reference Include="System.Xml" />
    </ItemGroup>
   <ItemGroup>
        <Compile Include="Program.cs" />
        <Compile Include="Properties\AssemblyInfo.cs" />
    </ItemGroup>
```

The references get added at build time, and do not show up in the project. You can see them in the references tree in visual studio with a NuGet icon.

### What doesn't work yet
Project types that's rely heavily on content packages and xdt transforms, won't work with project.json at the moment. The biggest ones are the MVC/Web API/Web Forms applications that pull in dependencies like jQuery, or expect nuget packages to modify web.config automatically for them.

For these project types you can move to using bower in combination with grunt/gulp.

### 3rd part projects/blogs related to this
A conversion script - https://github.com/wgtmpeters/nugetprojectjson
Marc Gravell's blog on DNX - http://blog.marcgravell.com/2015/11/the-road-to-dnxpart-3.html

