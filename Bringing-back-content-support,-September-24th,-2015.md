Content meeting notes September 24th 2015

* Goal: Add support for immutable package content and source files + .pp transforms
* Package's will use a new folder: shared\\{tfm} 
* Much of content is platform agnostic, an any folder will be needed.  "shared.any" ? (this was a previous convention suggestion)
* Users can exclude package content files for direct dependencies and transitive dependencies from referenced projects by adding the package to their project.json file with an additional property. NuGet will read this and update the lock file.
* Shared content and .pp files will be added to a new section in the lock file to be read by msbuild.
* MSBuild will apply .pp transforms. 
* PP transforms are needed to modify namespaces on source files.
* Source and .pp transforms could use a language sub folder that would be handled by msbuild.
* NuGet should handle making files from shared folders read only when opened in VS. This could be done by watching for document opened events and apply the setting if it comes from the global packages folder.
* Users who need to edit content will need to exclude the package content and install the content themselves. This could be done through a new nuget command. At this point NuGet would no longer manage the content for the user.
* MSBuild attributes for shared content will be listed in a new section of the nuspec file. This will allow users to embed or set copy local on content files. NuGet will parse the XML and add it to project.lock.json.
* Shared content will flow transitively to parent projects.

* In the future we want to add support for specifying a different root location per package, we haven't determined priority or design for this.
Discussion item - https://github.com/NuGet/Home/issues/1523