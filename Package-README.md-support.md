# Package README.md support

>The following work is in progress by the NuGet.org interns (@fayrose, @jarahsi, @laurenshen) and is expected to be released to preview by end of July 2017.
>
>This project was chosen based on feedback that authors and consumers want better package descriptions. Falling under the category of GitHub integrations, this is also related to the following issues:
>* https://github.com/NuGet/NuGetGallery/issues/2280
>* https://github.com/NuGet/NuGetGallery/issues/2925
>* https://github.com/NuGet/NuGetGallery/issues/4023

## Background

Currently, the package details page contains release notes, a small summary and description, version history, owners, authors, tags, copyright info, a few statistics relevant to the number of downloads, and other data such as the last time the package was updated and 1 link to a “project site.” Consumers have expressed that the package details page lacks the relevant information important in determining whether to download a package or not.

see: https://www.nuget.org/packages/bootstrap/

As part of improving the NuGet Gallery’s package details pages, we would like to display additional data such as repository statistics and ReadMe files.

>Note: We have decided that repository statistics will depend on GitHub OAuth integration. The idea is to only show statistics if owners' NuGet.org and GitHub accounts are linked and they are listed as admins/collaborators for the package repository. This is to maintain consumer trust in packages, by preventing association to arbitrary repositories.
>
>Note': GitHub integrations require associating repository URLs with packages. Some authors specify a repository URL in the packageURL metadata, while others prefer to use their project website. Support for repositoryURL metadata exists for NuGet packages created with Visual Studio 2017, and this spec suggests expanding that support to the Gallery.
>
>see:
>* https://docs.microsoft.com/en-us/nuget/guides/create-net-standard-packages-vs2017
>* https://docs.microsoft.com/en-us/nuget/schema/msbuild-targets#pack-target

## Goals

* ~~P0: Users can see the GitHub repository’s number of stars, number of open issues, and date of last commit on package detail pages.~~
* P0: An author can upload and edit a ReadMe.md file for display on package page for each specific version, with markdown support.
* ~~P0: An author can choose to opt out of the GitHub statistics display.~~
* P0: Differentiate between repository URL and project site URL, updating documentation to inform authors about the availability of source control URL and repository type in the nuspec.
* P1: An author can provide a GitHub link, select a commit, and we will pull in the ReadMe.md for a specific version and maintain respective ReadMe files for each version.
* ~~P2: Enable GitHub OAuth integration for authors to confirm access.~~

## Non-Goals

* Support for other platforms (i.e. VS Client, command line).

  *We want to be able to measure its success before investigating the potential to allow support for other platforms. Descriptions will be maintained for those viewing packages through the VS Client.*

* Redesign of other pages.

  *We will be adding more elements to the site and altering specific pages (as noted below, such as the Package Upload and Package Edit pages), but we won’t be altering any of the other pages’ elements.*

* Nuspec and client side.

  *We won’t be changing the nuspec or anything client side. The repo URL is already available in the nuspec, albeit undocumented, allowing us the benefits of the repo URL through the nuspec without the overhead of changing nuspec* parsing. A change in documentation and client outreach regarding the existence of this tag would allow package authors to take advantage of this.

* Other source control.

  *As of right now, we’ll only be supporting GitHub repositories, so we won’t have support for other source control platforms such as Bitbucket or TFS. In the future, however, we plan to expand to include these, if possible.*

## Solution

>Note: Originally we envisioned automatic opt-in of packages by populating the default README.md from Github repositories (using project or repository URL metadata). Now the vision is for README.md to be packed into the nupkg, similar to [README.txt](https://docs.microsoft.com/en-us/nuget/create-packages/creating-a-package#adding-a-readme-and-other-files). The scope of this work is limited to the Gallery, but could be extended to the client if the preview is a success.

### Repository URL

* Add repo URL column to packages table
* Existing packages
  * 1 time job in database to move (not copy) project URL to repo URL if GitHub
    * Prevents duplication
* Future packages (populate on package push)
  * First take from repo URL in nuspec
  * If no repo URL in nuspec, take from project URL if GitHub
  * May not maintain hierarchy in future, but necessary right now as there is no documentation about the existence of repo URL in the nuspec
* Recognize GitHub URL by hostname
* Alternative solutions
  a. Not surface repo URL, only use project URL if GitHub link (but packages such as Newtonsoft wouldn’t be able to use non-Git URL since they have an official project site)
  b. Adding a new repo type column to the database (since it exists in the nuspec

>Note the following changes to this section:
>
>Complete schema changes include:
>  * Update Packages table, adding RepositoryUrl and HasReadme columns
>  * Update PackageEdits table, adding RepositoryUrl and ReadmeState columns
>  * Update PackageHistories table, adding RepositoryUrl column
>
>Additionally, we would not modify existing data. Authors need to opt-in and explicitly upload a README.md through the Gallery for now.

### ~~GitHub Statistics~~ - section omitted

### ReadMe

#### UI
* Adding and editing ReadMe to package upload in Gallery (PackagesController)
  * Add new step for ReadMe upload, right before verify/last step
  * Browse and import from disk
    * Must be .MD file with size limit of 15 KB
    * If file not valid, display respective error message
    * Large textbox that will be populated with whatever was imported from disk, enabling editing
  * Default is large textbox populated with default example markdown text to encourage authors to create a ReadMe if they don’t have one
    * Textbox has limit of 10,000 characters
  * Preview of ReadMe in place (dynamically), transforming textbox to markdown
  * Question mark mouse over to give quick information on markdown syntax
  * Blurb about description versus ReadMe?
  * Version specific
* Editing and changing ReadMe in Gallery after package upload (PackagesController)
  * Same as adding ReadMe, except this will be located right under description box
* Viewing ReadMe on package details page
  * Markdown support
    * Markdown.com, 
  * Display ReadMe right below Release Notes
  * If more than ~20 lines, “See More” button
* Adding GitHub ReadMe to package upload in Gallery (P2)
  * If repo URL is populated from the nuspec or from the project URL as described above, latest GitHub ReadMe will be pulled
  * Latest GitHub ReadMe that was pulled will populate the description textbox, enables editing
  * Rest is similar to importing from disk/creating own
  * Store file upon package upload
  * Sync button for packages where repo URL is populated
  * Version specific, stored in container next to nupkg
* Editing and changing GitHub ReadMe to package upload in Gallery (P2)
  * Adding a sync button for packages where repo URL is populated, which will pull the latest ReadMe from repo URL and populate the large textbox with it, enabling edits
* Alternative Solutions
  * Allow authors can use GitHub ReadMe by typing a GitHub URL in a textbox, selecting a commit, and importing that ReadMe

>Note the following changes to this section:
>
>* UI design has fluctuated with the NuGet.org site redesign work. Upload from disk, URL or inline are still expected.
>* With the eventual goal of packing README.md into the nupkg, README.md size is only restricted by the package limit (250MB).
>* README.md will be displayed in addition to and under the description, instead of replacing it.
>* Project and/or repository URL will only be used as a hint path for authors to opt-in to README.md support
>* Repository URL and README.md changes will be queued with PackageEdits, same as with other package metadata
>* Stretch goal: support README.md from nupkg on push

#### Storage

* Blob storage
* Stores all imported .MD files, version specific
* Alternative Solutions
  * For ReadMes via GitHub link, pull directly from GitHub and then won’t need to store those files

>Note the following changes to this section:
>
> * README.md modifications will be applied in the backend (HandlePackageEdits).
> * With the eventual goal of packing README.md into the nupkg, we may apply modifications to the nupkg in blob storage in the future (requires client changes, which are out of scope)
> * Conversion from markdown to safe html should be done once, with both HTML and MD blobs created.
> * Blobs created for README.md support include:
>   * /readmes/active/id/normalizedVersion.md
>   * /readmes/active/id/normalizedVersion.html
>   * /readmes/pending/id/normalizedVersion.md
>   * /readmes/pending/id/normalizedVersion.html

## Testing and Monitoring

* Unit tests for source code
* Functional tests
  * Push package with GitHub URL through ApiController, verify ReadMe is available
  * Package with GitHub project URL in nuspec will show up with GitHub integrations
  * Package with only GitHub repo URL in nuspec will show up with GitHub integrations
  * Package with non-GitHub project URL and GitHub repo URL in nuspec will show up with GitHub integrations
  * Package with GitHub project URL and non-GitHub repo URL in nuspec will not show up GitHub integrations

>Note: Auditing will also need updating, but may be updated after preview.

## Execution Phases

>The following is expected by the end of July 2017:
>* Dependent schema and background (HandlePackageEdit) changes deployed to Production
>* Frontend support for README.md deployed to a Preview site