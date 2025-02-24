# Title

- Nikolche Kolev [nkolev92](https://github.com/nkolev92)
- Start Date (2021-07-06)
- GitHub Issue
  - [NuGet restore in VS should report progress to allow the project-system to better control design time build scheduling.](https://github.com/NuGet/Home/issues/9782)
  - [IVsPackageInstallerProjectEvents/IVsPackageInstallerEvents events not raised with PackageReference](https://github.com/NuGet/Home/issues/6872)

## Summary

<!-- One-paragraph description of the proposal. -->
When a PackageReference NuGet enabled project has been updated by NuGet, either a package install/uninstall/update, i.e.an assets file write, NuGet will fire an event to signify the start and end of the update.
The denoted interval will be the write `project.assets.json` and `projectName.projectExt.nuget.g.[props/targets]` files.
In addition, NuGet will provide a solution restore start/end event which will include a list of the relevant PackageReference projects.

## Motivation

<!-- Why are we doing this? What pain points does this solve? What is the expected outcome? -->
With SDK based projects, NuGet is tightly integrated with solution load & design time builds.
3 files are of interest for design time builds, `project.assets.json`, `projectName.projectExt.nuget.g.[props/targets]`.
When a restore happens, one or all of these files may be touched.
Given that these files are edited in quick succession, normally you'd end up getting only 1 design time build, but in some scenarios, the writes may take longer due to ambient reasons and cause extra design time builds.

## Explanation

### Functional explanation

<!-- Explain the proposal as if it were already implemented and you're teaching it to another person. -->
<!-- Introduce new concepts, functional designs with real life examples, and low-fidelity mockups or  pseudocode to show how this proposal would look. -->
Given that this is an extensibility feature, there's no NuGet experiences impact.
The functional impact matches what the consumers of this API use it for.

- For SDK based projects, the project-system will pause eager reevaluation while the writes of the `project.assets.json` and `projectName.projectExt.nuget.g.[props/targets]` are ongoing.

### Technical explanation

<!-- Explain the proposal in sufficient detail with implementation details, interaction models, and clarification of corner cases. -->

The API and the shape is fairly self explanatory.

```cs
// Copyright (c) .NET Foundation. All rights reserved.
// Licensed under the Apache License, Version 2.0. See License.txt in the project root for license information.

using System;
using System.Collections.Generic;
using System.Runtime.InteropServices;

namespace NuGet.SolutionRestoreManager
{
    /// <summary>
    /// NuGet PackageReference project update events.
    /// This API provides means of tracking project updates performance by NuGet on PackageReference projects, in particular updates to the assets file and nuget generated props/targets.
    /// All events are fired from a threadpool thread.
    /// </summary>
    [ComImport]
    [Guid("30CDDD0A-6901-482D-8CEF-6D798F1A99FC")]
    public interface IVsNuGetPackageReferenceProjectUpdateEvents
    {
        /// <summary>
        /// Raised when solution restore starts with the list of projects that will be restored.
        /// The list will not include all projects. Some projects may have been skipped in earlier up to date check, and other projects may no-op.
        /// </summary>
        /// <remarks>
        /// Just because a project is being restored that doesn't necessarily mean any actual updates will happen.
        /// Only PackageReference projects are included in this list.
        /// No heavy computation should happen in any of these methods as it'll block the NuGet progress.
        /// </remarks>
        event SolutionRestoreEventHandler SolutionRestoreStarted;

        /// <summary>
        /// Raised when solution restore finishes with the list of projects that were restored.
        /// The list will not include all projects. Some projects may have been skipped in earlier up to date check, and other projects may no-op.
        /// </summary>
        /// <remarks>
        /// Just because a project is being restored that doesn't necessarily mean any actual updates will happen.
        /// Only PackageReference projects are included in this list.
        /// No heavy computation should happen in any of these methods as it'll block the NuGet progress.
        /// </remarks>
        event SolutionRestoreEventHandler SolutionRestoreFinished;

        /// <summary>
        /// Raised when particular project is about to be updated.
        /// This means an assets file or a nuget temp msbuild file write (nuget.g.props or nuget.g.targets). The list of updated files will include the aforementioned.
        /// If a project was restore, but no file updates happen, this event will not be fired.
        /// </summary>
        /// <remarks>
        /// No heavy computation should happen in any of these methods as it'll block the NuGet progress.
        /// </remarks>
        event ProjectUpdateEventHandler ProjectUpdateStarted;

        /// <summary>
        /// Raised when particular project update has been completed.
        /// This means an assets file or a nuget temp msbuild file write (nuget.g.props or nuget.g.targets). The list of updated files will include the aforementioned.
        /// If a project was restore, but no file updates happen, this event will not be fired.
        /// </summary>
        /// <remarks>
        /// No heavy computation should happen in any of these methods as it'll block the NuGet progress.
        /// </remarks>
        event ProjectUpdateEventHandler ProjectUpdateFinished;
    }

    /// <summary>
    /// Defines an event handler delegate for PackageReference solution restore start and end.
    /// </summary>
    /// <param name="projects">List of projects that will run restore. Never <see langword="null"/>.</param>
    public delegate void SolutionRestoreEventHandler(IReadOnlyList<string> projects);

    /// <summary>
    /// Defines an event handler delegate for project updates.
    /// </summary>
    /// <param name="projectUniqueName">Project full path. Never <see langword="null"/>. </param>
    /// <param name="updatedFiles">NuGet output files that may be updated. Never <see langword="null"/>.</param>
    public delegate void ProjectUpdateEventHandler(string projectUniqueName, IReadOnlyList<string> updatedFiles);
}
```

A few other notes:

- `SolutionRestoreStarted` and `SolutionRestoreFinished` events will only be called when a solution restore happens, as a response to a solution/project load, or a user issued restore. Package installation through PM UI, PMC or any of the NuGet extensibility APIs *will never* call this event.
- `ProjectUpdateStarted` and `ProjectUpdateFinished` may be called without corresponding `SolutionRestoreStarted` and `SolutionRestoreFinished` events, if a package installation has been triggered through PM UI, PMC or any of the NuGet extensibility APIs.

## Drawbacks

<!-- Why should we not do this? -->

- These events are meant to provide a means for better coordinating work for partner teams.

## Rationale and alternatives

<!-- Why is this the best design compared to other designs? -->
<!-- What other designs have been considered and why weren't they chosen? -->
<!-- What is the impact of not doing this? -->
- Why not provide events include these events for packages.config based projects as well?
`packages.config` and `PackageReference` projects differ fundamentally in the way they deal with package changes.
  - In `packages.config` projects, NuGet writes to the *project* files, and NuGet references are equivalent to any other new references.
  - In `PackageReference` projects, the contract is expressed through the assets file. An assets file change may mean that 10 packages have changes.
  - Components such as test explorer, need to know when packages for a project have changed to scan for adapters.
  - For `packages.config` projects, one can use [`IVsPackageInstallerEvents`](https://docs.microsoft.com/en-us/nuget/visual-studio-extensibility/nuget-api-in-visual-studio#ivspackageinstallerevents-interface). There's no PackageReference equivalent.
  - There is no consistent API shape that could fit *both* packages.config and PackageReference projects. As such we're choosing not to go this direction.
  - A rejected proposal was the following:
  
```cs
namespace NuGet.SolutionRestoreManager
{
    /// <summary>
    /// NuGet project update events.
    /// Architecturally, packages.config and PackageReference projects differ in the way package updates are processed.
    /// This interface is meant to provide a single API of interest for components wanting to listen to *all* project updates by NuGet.
    /// </summary>
    [ComImport]
    [Guid("30CDDD0A-6901-482D-8CEF-6D798F1A99FC")]
    public interface IVsNuGetProjectUpdateEvents
    {
        /// <summary>
        /// Raised when solution restore starts with the list of projects that will be restored.
        /// The list will not include all projects. Some projects may have been skipped in earlier up to date check, and other projects may no-op.
        /// </summary>
        /// <remarks>
        /// Just because a project is being restored that doesn't necessarily mean any actual updates will happen.
        /// Only PackageReference projects are includede in this list.
        /// </remarks>
        event SolutionRestoreEventHandler SolutionRestoreStarted;

        /// <summary>
        /// Raised when solution restore finishes with the list of projects that were restored.
        /// The list will not include all projects. Some projects may have been skipped in earlier up to date check, and other projects may no-op.
        /// </summary>
        /// <remarks>
        /// Just because a project is being restored that doesn't necessarily mean any actual updates will happen.
        /// Only PackageReference projects are includede in this list.
        /// </remarks>
        event SolutionRestoreEventHandler SolutionRestoreFinished;

        /// <summary>
        /// Raised when particular project is about to be updated.
        /// For PackageReference projects, this means an assets file or a nuget temp msbuild file write (nuget.g.props or nuget.g.targets). The list of updated files will include the aforementioned.
        /// For packages.config projects, this means a single package is installed/unistall/unistalled. The list of updated files may include the path of the package that was changed.
        /// </summary>
        event ProjectUpdateEventHandler ProjectUpdateStarted;

        /// <summary>
        /// Raised when particular project update has been completed.
        /// For PackageReference projects, this means an assets file or a nuget temp msbuild file write (nuget.g.props or nuget.g.targets). The list of updated files will include the aforementioned.
        /// For packages.config projects, this means a single package is installed/unistall/unistalled. The list of updated files may include the path of the package that was changed.
        /// </summary>
        event ProjectUpdateEventHandler ProjectUpdateFinished;
    }

    /// <summary>
    /// Defines an event handler delegate for PackageReference solution restore start and end.
    /// </summary>
    /// <param name="projects">List of projects that will run restore. Never <see langword="null"/>.</param>
    public delegate void SolutionRestoreEventHandler(IReadOnlyList<string> projects);

    /// <summary>
    /// Defines an event handler delegate for project updates.
    /// </summary>
    /// <param name="projectUniqueName">Project full path. Never <see langword="null"/>. </param>
    /// <param name="updatedFiles">NuGet output files that may be updated. Never <see langword="null"/>.</param>
    public delegate void ProjectUpdateEventHandler(string projectUniqueName, IReadOnlyList<string> updatedFiles);
}
```

- Why not provide custom events for package installation/uninstallation like in packages.config projects? Why not implement IVSPackageInstallerEvents in PackageReference.
  - PackageReference based projects have many means of installing packages. With SDK based project, a project file edit could lead to a few package installation or uninstallations. Keeping track of the exact packages that were installed and uninstalled is not often relevant, as many of the extensions that listen to these updates call `GetInstalledPackages` whenever there is a change.
- Adding the events in the [IVsSolutionRestoreStatusProvider](https://github.com/NuGet/NuGet.Client/blob/dev/src/NuGet.Clients/NuGet.VisualStudio/SolutionRestoreManager/IVsSolutionRestoreStatusProvider.cs)
  - This API provides a means to detect if a restore is running, and provide restore updates in this API would be appropriate. The new API is just more specific. Given the unlikely usage of that API, there's no real benefit to adding it there.

## Prior Art

<!-- What prior art, both good and bad are related to this proposal? -->
<!-- Do other features exist in other ecosystems and what experience have their community had? -->
<!-- What lessons from other communities can we learn from? -->
<!-- Are there any resources that are relevant to this proposal? -->

- [`IVsPackageInstallerEvents`](https://docs.microsoft.com/en-us/nuget/visual-studio-extensibility/nuget-api-in-visual-studio#ivspackageinstallerevents-interface) - Provides events for the installation of packages in packages.config projects.
- [`IVsSolutionRestoreProvider`](https://github.com/NuGet/NuGet.Client/blob/dev/src/NuGet.Clients/NuGet.VisualStudio/SolutionRestoreManager/IVsSolutionRestoreStatusProvider.cs) - Provides a means to detect if a restore is running.

## Unresolved Questions

<!-- What parts of the proposal do you expect to resolve before this gets accepted? -->
<!-- What parts of the proposal need to be resolved before the proposal is stabilized? -->
<!-- What related issues would you consider out of scope for this proposal but can be addressed in the future? -->

## Future Possibilities

<!-- What future possibilities can you think of that this proposal would help with? -->
