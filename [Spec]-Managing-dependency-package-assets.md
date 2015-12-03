This spec covers the design for including and excluding parts of dependency packages. 

Include flags are defined on dependency edges, not on the packages themselves. Projects and packages are treated the same with the exception that direct project references will include content by default. Packages receive an intersection of the edge flags when walking down the dependency graph. The set of flags for each package target id is the union of the flags applied from the walk. Flags for direct project dependencies will override all other flags. This is explained further in the examples.

### Include Flags

|Flag|Description|
|-------------|----------------------------------------------------|
|contentFiles|Content v2 items|
|runtime|Includes the Runtime, Resources, and FrameworkAssemblies sections of the target|
|compile|Lock file compile section of the target|
|build|MSBuild targets and properties in the build folder|
|native|Native folder|
|none|Empty|
|all|All flags|

### Project.json 

Excluding content files from a package
```json
{
  "dependencies": {
    "packageA": {
      "version": "1.0.0",
      "exclude": "contentFiles"
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
      "include": "runtime"
    }
  }
}
```

### Nuspec

Content v2 is turned off for transitive dependencies by default. Packages must opt in to allowing content to flow up from dependency packages. By specifying *all* as the include flag *contentFiles* will be turned on.
```xml
<dependencies>
  <group>
    <dependency id="packageB" version="1.0.0" include="all" />
  </group>
</dependencies>
```

Dependency items may also provide an exclude property instead of listing out each part of the package to include. By defining ``exclude="build"`` the dependency packageB will not add msbuild targets and prop files to the project.
```xml
<dependencies>
  <group>
    <dependency id="packageB" version="1.0.0" exclude="build" />
  </group>
</dependencies>
```

### Multiple flags

The *include* and *exclude* properties for both project.json and nuspec dependencies support comma delimited flags.

``"build,native,runtime"``

### Flag precedence

Exclude takes precedence over include. 

``"include": "runtime,compile", "exclude": "compile"`` is equivalent to ``"include": "runtime"``

### Content v2 behavior

Content v2 is disabled by default for all package dependencies except for those referenced directly by the project in project.json. Transitive packages coming from other packages and from other projects will not bring in content unless the package or project has opted into it using the needed include.

### Suppress Parent
Projects may have dependencies that are not needed by consumers of the project. For example if ProjectA depends on a package *X* which has content files, ProjectB should not transitively depend on package *X* since the content was compiled into ProjectA and will come from that assembly.

To solve this the includes on dependency edges can be thought of in two ways.
* Behavior that applies to myself (for consumers of this project)
* Behavior that applies to my dependencies (for this project)

To define additional excludes for consumers of the project the ``suppressParent`` property is used. In the below example the content only package is used by the project, but consumers of the project do not get the dependency.

```json
{
  "dependencies": {
    "contentOnlyPackage": {
      "version": "1.0.0",
      "suppressParent": "all"
    }
  }
}
```

A build time only dependency can be created by excluding the dependency for the consumers of the project. When this project is packed packageA will not be included as a dependency.

```json
{
  "dependencies": {
    "packageA": {
      "version": "1.0.0",
      "suppressParent": "all"
    }
  }
}
```

A private dependency can be created by excluding the compile section for consumers of the project.

```json
{
  "dependencies": {
    "packageA": {
      "version": "1.0.0",
      "suppressParent": "compile"
    }
  }
}
```

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
When include="contentFiles" is used on all dependency edges in the nuspec content will flow transitively for packages A, B, and C.

```
Project
 |-(all)-> A -(-build)-> B -(-compile)-> C
```
C will have both build and compile excluded since both of these were blocked by parent packages.

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
In the reverse of the previous example, if the project excludes the build folder from B it will override the other flags and B will exclude the build section from the target. This allows users to force package behaviors when needed.