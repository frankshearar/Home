# MSBuild restore target

As part of the move from xproj to MSBuild package restore will move to an MSbuild target. Previous restore entry points such as *nuget.exe restore* and *dotnet.exe restore* will start using this target to restore .NET Core projects.

## Restoring packages using the restore target

The restore target will gather all msbuild data and pass it into the restore task in NuGet.Build.Tasks, this task is a wrapper for the existing restore command from NuGet.Commands.

1. Read all project to project references
1. Read the project properties to find the intermediate folder and target frameworks
1. Pass msbuild data to NuGet.Build.Tasks.dll
1. Run restore
1. Download packages
1. Write assets file, targets, and props

## Restore properties
Additional restore settings may come from MSBuild properties.

| Property | Multiple | Description |
| -------- | ------- | ----------- |
| RestoreSources | ``;`` delimited | List of package sources |
| RestorePackagesPath | | User packages directory path |
| RestoreDisableParallel | | Limit downloads to one at a time |
| RestoreConfigFile | | nuget.config file |
| RestoreNoCache | |  Avoid using the web cache if set to *true* |
| RestoreIgnoreFailedSource | | Ignoring failing or missing package sources if set to *true* |
| RestoreTaskAssemblyFile | | Path to NuGet.Build.Tasks.dll |
| RestoreGraphProjectInput | ``;`` delimited  | list of projects to restore, this should contain absolute paths |
| RestoreOutputPath | | Output directory, by default the obj folder is used |

**Example**

```msbuild
<PropertyGroup>
   <RestoreIgnoreFailedSource>true</RestoreIgnoreFailedSource>
<PropertyGroup>
```

## Restore outputs

Restore will create the following files in the build obj folder

| File | Description |
| ---- | ----------- |
| project.assets.json | Previously project.lock.json |
| project.generated.targets | References to msbuild targets contained in packages |
| project.generated.props | References to msbuild props contained in packages |

### Multiple asset files per project

project.assets.json will be placed in the intermediate folder for a specific target framework. Instead of containing all combinations of frameworks and RIDs it will contain a single framework.

The restore task will read target frameworks from the project file and then restore for all frameworks.

RIDs will be passed into msbuild, these will then flow through to NuGet restore and will create additional runtime graphs in the assets file.

## .dg file format

Data collected from MSBuild is put into a .dg file. This is kept in memory but can be written to a file and passed to nuget.exe restore as an input.

The file format is basic and consists of ``|`` delimited lines that are prefixed with a character to denote the entry type and then a ``:``.

**Example of a project with one reference**
```
#:/src/myProj.csproj
$:netstandard1.6
+:RestoreOuputType|netcore
+:RestoreOuputPath|/src/myProj/obj/
+:RestoreFrameworks|netstandard1.6
=:/src/myProj.csproj|/src/myChildProj.csproj
#:/src/myChildProj.csproj
+:RestoreOuputType|netcore
+:RestoreOuputPath|/src/myChildProj/obj/
+:RestoreFrameworks|netstandard1.3
```

| Character | Description |
| --------- | ----------- |
| **#** | Entry point, these projects will be restored. This also marks the beginning of a section for a project, all entries below this are considered part of that project until another section is encountered. |
| **$** | Restore output separator, this allows for multiple restore outputs. Format ``Comment`` |
| **+** | Property value. Format ``Name|Value`` |
| **=** | Project to project reference. Format ``Parent|Child`` |

The project to project entries represent edges in the project graph, restore adds these edges to the restore graph along with the package references to create a full graph of all dependencies for the project when building the lock file.

Properties in the .dg file

| Property  | Description |
| --------- | ----------- |
| RestoreOutputPath | Output location, this is typically the intermediate folder |
| RestoreOutputType | Restore type, if set to netcore assets will be placed in the output path with the new names. |
| RestoreFrameworks | A set of target frameworks to restore for. |
| RestoreRuntimes   | A set of RIDs to restore for, these will be combined with all target frameworks. |

## Open issues

* Is it possible to get the same behavior between msbuild and Visual Studio
* msbuild /t:restore solution.sln needs a target in the metaproj
* should RID restores go into different files such as project.assets.{RID}.json