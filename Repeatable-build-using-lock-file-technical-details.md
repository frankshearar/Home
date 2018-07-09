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

## Enabling the use of lock file

There will be multiple ways a customer can opt into this feature and start to use a project level lock file.

* A lock file is present in the context of the project.
* If the MSBuild property RestoreWithLockFile is set in the context of the project.

