This spec covers the design for including and excluding parts of dependency packages. 

Include flags are defined on dependency edges, not on the packages themselves. Projects and packages are treated the same with the exception that direct project references will include content by default. Packages receive an intersection of the edge flags when walking down the dependency graph. The set of flags for each package target id is the union of the flags applied from the walk. This is explained further in the examples.

### Include Flags

|Flag|Description|
|-------------|----------------------------------------------------|
|ContentFiles|Content v2 items|
|Runtime|Includes the Runtime, Resources, and FrameworkAssemblies sections of the target|
|Compile|Lock file compile section of the target|
|Build|MSBuild targets and properties in the build folder|
|Native|Native folder|
|Dependencies|Package dependencies|
|None|Empty|
|All|All flags|

|Dependency edge|Default|
|-------------|----------------------------------------------------|
|direct project reference|All flags including contentFiles|
|package -> package|All flags except contentFiles|


### Project.json 

Excluding content files from a package
```json
{
  "dependencies": {
    "packageA": {
      "version": "1.0.0",
      "excludeFlags": "contentFiles"
    }
  }
}
```

Including only runtime components for a package:
```json
{
  "dependencies": {
    "packageA": {
      "version": "1.0.0",
      "includeFlags": "runtime"
    }
  }
}
```

### Nuspec

Content v2 is turned off for transitive dependencies by default. Packages must opt in to allowing content to flow up from dependency packages. By specifying All as the include flag *contentFiles* will be turned on.
```xml
<dependencies>
  <group>
    <dependency id="packageB" version="1.0.0" includeFlags="all" />
  </group>
</dependencies>
```

If only the content file are needed from a dependency a package may exclude all other pieces and just include content. This will also exclude dependencies for that package.
```xml
<dependencies>
  <group>
    <dependency id="packageB" version="1.0.0" includeFlags="contentFiles" excludeFlags="all" />
  </group>
</dependencies>
```

### Content v2 behavior

Content v2 is disabled by default for all package dependencies except for those referenced directly by the project in project.json. Transitive packages coming from other packages and from other projects will not bring in content unless the package or project has opted into it using the needed includeFlags.


### Examples

```
Project
 |-(all)-> A -(default)-> B -(default)-> C
 |-(all)-> C
```
In the above example the project will reference content included in packages A and C. Content will not be included from B since A does not declare it in the dependency reference.

```
Project
 |-(all)-> A -(+contentFiles)-> B -(+contentFiles)-> C
```
When includeFlags="contentFiles" is used on all dependency edges in the nuspec content will flow transitively for packages A, B, and C.

```
Project
 |-(all)-> A -(-build)-> B -(-compile)-> C
```
C will have both build and compile excluded since both of these were block by parent packages.

```
Project
 |-(all)-> A -(-build)-> B
 |-(all)-> B
```
A does not use the build targets from B and excludes it, however the project has a direct reference to B which brings in all parts of B including the build folder. The exclude defined in A has no effect here since it would block B from being used.

```
Project
 |-(all)-> A -(+build)-> B
 |-(-build)-> B
```
In the reverse of the previous example, if the project excludes the build folder from B it will still be included since another package has included it. Users can force B to behave as defined by the project by excluding dependencies for A.