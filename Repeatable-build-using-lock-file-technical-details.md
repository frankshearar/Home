Status: **Incubation**

## Issue

The discussion around this spec is tracked here - **Enable repeatable package restore using lock file [Spec](https://github.com/NuGet/Home/wiki/Enable-repeatable-package-restore-using-lock-file)** 

## Introduction

This specification provides technical details on how are we going to achieve repeatable builds for PackageReference based projects using project lock file.

## Description of a lock file

Purpose of a lock file is to help lock down NuGet package's resolved version so that NuGet restore graph doesn't change if there is no explicit user change in the project or it's dependencies. Keeping that in mind, following will be the content of a lock file.

* Lock file version.
* List of NuGet packages dependencies for each TFM of the project. Each dependency could of type:
  * `Direct` - NuGet package dependency directly added into the project.
  * `Transitive` - NuGet package dependency transitively coming into the project either through direct package dependency or from another project dependency.
  * `Project` - Project (P2P) reference.
* Following be the properties of each NuGet package dependency:
  * Package name
  * Package requested version (only applicable for `Direct` dependencies)
  * Package resolved version
  * Type (as mentioned above)
  * SHA512 of the package
  * Dependencies of the current package in form of `<Id>:<Version>` (as mentioned in the package nuspec file)

## Enabling the use of lock file

There will be multiple ways a customer can opt into this feature and start to use a project level lock file.

* A lock file is present in the context of the project.
* If the MSBuild property RestoreWithLockFile is set in the context of the project.

## Usage of lock file wrt `restore`

This is where all the technical details lies for how are we going to consume/ author this lock file during restore.

* As of now, we won't be consuming this lock file during restore noop check so this lock file will only come into picture if restore noop check has failed and we continue with the normal restore.
* First thing is to check if there is an existing project lock file to be consumed for the current restore. And if there is then,
  * Read the lock file and construct in-memory lock object similar to LockFile instance.
  * Then check if lock file is still valid, meaning if it's not [out of sync](https://github.com/NuGet/Home/wiki/Enable-repeatable-package-restore-using-lock-file#out-of-sync) wrt user changes.
  * If still valid, then pass this lock file data to further down the stream to be used as part of generating restore graph and making sure that all packages are resolved to the versions define in lock file.
  * If not valid, then discard this lock file and continue with regular restore.
* Finally once the restore is completed, check if user has opt into this feature, then
  * if nothing changed in the existing file, then don't over-write it.
  * else, write the project lock file.

## Validating the lock file

* Check all the `direct` NuGet dependencies and their requested versions still match with the user's input.
* Check all the `project` dependencies from lock file still match with project's P2P references and all their direct dependencies are still the same.

## Open Items

* lock file name `packages.lock.json` or `nuget.lock.json` or <something-else>.
* Finalize UI option to run restore with reevaluate restore graph even with a lock file.