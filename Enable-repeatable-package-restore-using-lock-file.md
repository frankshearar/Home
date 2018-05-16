* Status: Incubation
* Author(s): [Anand Gaurav](https://github.com/anangaur) ([@adgrv](https://twitter.com/adgrv))

### Requirements
Refer to the spec:[[Centrally managing NuGet packages]] for the list of requirements and summary of the solution. This spec details out the solution for enabling repeatable package restore (build) using a lock file.

*For central management of nuget package versions, refer to the spec: [[Centrally managing NuGet package versions]]

### Solution Details
* If a central package version management file (default `packages.props` file is present, NuGet will not just use this file for manage package versions, but it will also create a lock file - `packages.lock.json` in the same folder as the central package version management file. This file will have the full package closure - direct and transitive across the projects in a repo.
* If you `restore` in a project's context, NuGet will refer the `packages.lock.json` to get the package closure and restore those packages.
* If there is any of the following discrepancies while resolving package dependencies for a project, `restore` will error out.
  * `PackageReference` in project file does not have a corresponding entry in the `packages.lock.json` and/or `packages.props`
  * Version mismatch between `packages.lock.json` and `packages.props`
* In order to update the lock file with new restore graph, you can run `restore` with option `update-lock-file`:
  ```
  > dotnet restore --update-lock-file
  ```
* Specifying a `<Package>` node in the `packages.props` does not mean the lock file will also contain this package information. The lock file is updated on a package reference addition to a project. 

*packages.lock.json*
```
{	
  "version": 1.0,	
  "metadata1":"value1",
  ...other metadata fields...
  "dependencies": {	
    "netcoreapp2.0": {	
      "Contoso.Base": {	
        "type": "direct",	
        "requested": "3.0.0",	
        "resolved": "3.0.0",
        "integrity":"SHA512-#fVXsnMP2Wq84VA533zj0a/Et+QoLoeNpVXsnMP2Wq84l+hsUxfwunkbqoIHIvpOqwQ/+HIvprVKs+QOihnkbqod=="		
      }	
  ...	
```
