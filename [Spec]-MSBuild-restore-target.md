# MSBuild restore target

## Restoring packages using the restore target

The restore target will gather all msbuild data and pass it into the restore task in NuGet.Build.Tasks, this task is a wrapper for the existing restore command from NuGet.Commands.

1. Read all project to project references
1. Read the project properties to find the intermediate folder and target frameworks
1. Pass msbuild data to NuGet.Build.Tasks.dll
1. Run restore
1. Download packages
1. Write assets file, targets, and props


## .dg file format

Data collected from MSBuild is put into a .dg file. This is kept in memory but can be written to a file and passed to nuget.exe restore as an input.

The file format is basic and consists of ``|`` delimited lines that are prefixed with a character to denote the entry type and then a ``:``.

**Example of a project with one reference**
```
#:/src/myProj.csproj
+:RestoreOuputPath|/src/myProj/obj/
=:/src/myProj.csproj|/src/myChildProj.csproj
#:/src/myChildProj.csproj
+:RestoreOuputPath|/src/myChildProj/obj/
```

| Character | Description |
| --------- | ----------- |
| **#** | Entry point, these projects will be restored. This also marks the beginning of a section for a project, all entries below this are considered part of that project until another section is encountered. |
| **+** | Property value. Format ``Name|Value`` |
| **=** | Project to project reference. Format ``Parent|Child`` |

The project to project entries represent edges in the project graph, restore adds these edges to the restore graph along with the package references to create a full graph of all dependencies for the project when building the lock file.

Properties such as *RestoreOutputPath* are used by restore to determine where the assets file, generated targets, and generated props.