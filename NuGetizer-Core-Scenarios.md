## Portable Library

Alice wants to create a new library that targets multiple platforms.

She opens Visual Studio, goes to _File > New Project_, and selects _NuGet Package_. A wizard opens prompting for basic package metadata such as ID, author and description. The wizard also offers her a choice of which platforms she would like to target, and whether she would like to use per-platform implementations (bait-and-switch) or a single implementation for all platforms (PCL). She chooses to create a PCL, and chooses to target Android, iOS and UWP.

A PCL project is created targeted for the platforms that Alice selected, with project options set to build a NuGet package with the metadata she provided. Alice right-clicks on the project and chooses _Create NuGet Package_, and a nupkg is built that includes the PCL library in the correct lib subdirectory.

## Platform-Specific Implementation

Bob has created a PCL (or netstandard) project that builds a NuGet package. He would like to create a platform specific implementation of some of the functionality for iOS.

He right-clicks on the PCL project and chooses _Add Platform Implementation_. A dialog opens asking him what platforms he would like to add, and whether he would like to factor a shared project out of the PCL. He chooses to add the iOS platform and to use the shared project.

A new iOS project is created. A NuGet project is created that uses the metadata from the PCL project, and references the iOS and PCL projects. The items in the PCL project are factored out into a new shared project that is referenced by both the PCL project and the iOS project.

Bob implements iOS specific functionality using files in the iOS project and `#ifdefs` inside the shared project.

Bob right-clicks on the NuGet project and chooses _Create NuGet Package_, and a nupkg is built that includes both the PCL library and the iOS library.

## Bait-and-Switch

Corey has created a NuGet Package project that references iOS, Android and UWP library projects that share code via a shared project. They do not have a PCL project as they do not have a portable implementation, but they would like to allow their NuGet to be referenced from PCL projects.

They open the _Project Options_ for the NuGet Package project and go to the _Reference Assemblies_ tab. There they see a section with checkboxes to generate reference assemblies for the frameworks of any of the referenced library projects, the netstandard framework, and any of the PCL profiles that are compatible with the referenced frameworks. They check the netstandard checkbox.

When Corey builds the NuGet Package project, the build process automatically generates a netstandard reference assembly from the intersection of the iOS, Android and UWP implementations and includes it in the nupkg.

## Package Existing Project

Dennis would like to create a single-platform NuGet package from an existing iOS library.

He opens the _Project Options_, goes to the _NuGet_ panel, and fills out the package metadata. He right-clicks on the project and chooses _Create NuGet Package_, and a nupkg is built.

## New Metapackage

Edith has a solution that contains several related projects that produce NuGet packages. She would like to create a metapackage - a package that depends on them but contains no libraries or content.

She opens the _File > New Project_ dialog, and selects NuGet Package. A wizard opens prompting for package metadata. She fills it out and clicks OK.

A new NuGet Package project is created. Edith references her existing NuGet Package project from the new NuGet Package project, and when she builds it, a NuGet metapackage is built that depends on the packages built by her existing NuGet Package projects.
