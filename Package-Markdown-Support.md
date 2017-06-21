# Background
Currently, the package details page contains release notes, a small summary and description, version history, owners, authors, tags, copyright info, a few statistics relevant to the number of downloads, and other data such as the last time the package was updated and 1 link to a “project site.” Consumers have expressed that the package details page lacks the relevant information important in determining whether to download a package or not.

As part of improving the NuGet Gallery’s package details pages, we would like to display additional data such as repository statistics and ReadMe files.

*The ultimate goal is to support README.md inside the nupkg, similar to [README.txt](https://docs.microsoft.com/en-us/nuget/create-packages/creating-a-package#adding-a-readme-and-other-files). We will initially add Gallery support for inserting the README.md into the package after upload, but can later extend support to the client during packing.*

## Goals
- [P0]: An author can associate a repository URL with their package, separate from the existing project URL.
- [P0]: An author can upload a README.md file for display on package page for each specific version, with markdown support.
  - Upload file from client
  - Download file from (non-persisted) URL, defaulting to the README.md from the repository URL.
- [P0]: An author can choose to opt-in to the README.md feature for future package pushes (pending clarification).
- [Stretch]: NuGet client support for packing with README.md.
- [Stretch]: GitHub OAuth support and statistics integration.

## Non-Goals
- Dynamic updates to README.md from GitHub.
- Redesign of other pages (limited to Package Verify/Edit/Details).
- NuSpec changes (aside from repository URL).
- Other repository types (limited to GitHub).

## Solution

### Schema: Repository URL

  Add repository URL to the Packages (version) and PackageEdits table. Note that there is some support for this in clients (csproj, but not nuget.exe)
  - see: [ManifestReader.cs](https://github.com/NuGet/NuGet.Client/blob/dev/src/NuGet.Core/NuGet.Packaging/PackageCreation/Authoring/ManifestReader.cs)
  - see: [PackageSpec.cs](https://github.com/NuGet/NuGet.Client/blob/dev/src/NuGet.Core/NuGet.ProjectModel/PackageSpec.cs)

### User Interface

- Upload/Verify Package and Edit Package views
  - Add textbox for repo URL
  - Add opt-in checkbox for package registration (pending clarification)
  - Add UI for uploading README.md from disk or URL

- Package Details view
  - Support for README.md rendering, if found
  - Display under (pending: and in addition to) description.

### Job and Storage

Keeping the ultimate goal in mind, the README.md upload/edit should update the nupkg in cloud storage to include the README.md. Eventually client will support packing the nupkg originally with the README.md. The README.md should also be uploaded to a separate blob as a Gallery optimization for rendering the markdown.

- Gallery Trigger
  - Creates PackageEdit for README (pending: do we need README.md bool column?)
  - Uploads PackageEdit blob for README.md edit
  - Cancel of PackageEdit should delete pending README.md edit blob

- NuGet.Jobs/HandlePackageEdits
  - Supports repository URL updates; updates nuspec
  - Supports README.md updates; replaces blob and updates nupkg