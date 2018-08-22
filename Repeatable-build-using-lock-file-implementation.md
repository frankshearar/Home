* Status: **Incubation**
* Author(s): [Ashish Jain](https://github.com/jainaashish)

## Issue

The discussion around this spec is tracked here - **Enable repeatable package restore using lock file [Spec](https://github.com/NuGet/Home/wiki/Enable-repeatable-package-restore-using-lock-file)** 

## Introduction

This specification provides technical details on how are we going to achieve repeatable builds for PackageReference based projects using project lock file.

## Lock file name

By default lock file name will be `packages.lock.json` file which will be created in project root directory. But there will be multiple ways to over-write it.

* Similar to `packages.config` file, customers can also include `projectName` in lock file name to have multiple lock files in same folder for multiple projects. Consumers will need to replace space with `_` in project name and append in lock file name, something like `packages.<project_name>.lock.json`.
* By setting `NuGetLockFilePath` MSBuild property or passing `--lock-file` command line parameter with `dotnet.exe`. This option also allows to over-write the default path since this is the absolute file path to the lock file, something like `*lock.json`

## Description of a lock file

Purpose of a lock file is to help lock down NuGet package's resolved version so that NuGet restore graph doesn't change if there is no explicit user change in the project or it's dependencies. Keeping that in mind, following will be the content of a lock file.

* Lock file version.
* List of NuGet packages dependencies for each TFM of the project. When there are `RuntimeIdentifiers (Rids)` defined in the project, then each `{TFM}/{RID}` graph will only have those dependencies which are different than original `{TFM}` graph. Each dependency could of type:
  * `Direct` - NuGet package dependency directly added into the project.
  * `Transitive` - NuGet package dependency transitively coming into the project either through direct package dependency or from another project dependency.
  * `Project` - Project (P2P) reference.
* Following be the properties of each NuGet package dependency:
  * Package name
  * Package requested version (only applicable for `Direct` dependencies)
  * Package resolved version
  * Type (as mentioned above)
  * SHA512 hash of the package
  * Dependencies of the current package in form of `<Id>:<Version>` (as mentioned in the package nuspec file)

## Enabling the use of lock file

There will be multiple ways a customer can opt into this feature and start to use a project level lock file.

* A lock file is present in the context of the project. So customer can simply create a blank `packages.lock.json` file and opt into this feature.
* If the MSBuild property RestoreWithLockFile is set in the context of the project.

## Usage of lock file wrt `restore`

This is where all the technical details lies for how are we going to consume/ author this lock file during restore.

* As of now, we won't be consuming this lock file during restore noop check so this lock file will only come into picture if restore noop check has failed and we continue with the regular restore.
* First thing is to check if restore is being run with `--reevaluate` flag, if yes then no need to read lock file instead continue with the regular restore. otherwise,
* If there is an existing project lock file to be consumed for the current restore then,
  * Read the lock file and construct in-memory lock object similar to LockFile instance.
  * Then check if lock file is still valid, meaning if it's not [out of sync](https://github.com/NuGet/Home/wiki/Enable-repeatable-package-restore-using-lock-file#out-of-sync) wrt user changes.
  * If still valid, then pass this lock file data to further down the stream to be used as part of generating restore graph and making sure that all packages are resolved to the versions define in lock file.
    * if we fail to resolve any package with locked version from configured sources, then restore will fail with appropriate error message.
  * If existing lock file not valid, then
    * if user has denied updating existing lock file by setting `RestoreLockedMode` property to true, then throw appropriate restore error and bail.
    * else, discard this lock file and continue with regular restore.
* Finally once the restore is completed, check if user has opt into this feature, then
  * if nothing changed in the existing lock file, then don't over-write it.
  * if no existing lock file, then write it out.
  * else, check if user has allowed to overwrite the existing lock file, then update it.
* If lock file was used to restore, then there will be a post-restore step to validate all resolved package's SHA512 hash matches with lock file and making sure that all the same packages are restored which were locked. If there is any mis-match then restore will fail with appropriate error message.

## Validating the lock file

* Check all the `direct` NuGet dependencies and their requested versions still match with the user's input.
* Check all the `project` dependencies from lock file still match with project's P2P references and all their direct dependencies are still the same.

## When to update the existing lock file

* Install/ Update/ Uninstall - These actions (either from VS UI or `dotnet.exe add package`) will update the lock file, if required.
* Restore - By default, if the lock file is out of sync, restore command will update the lock file with the latest resolved closure of packages. But there will be an explicit MSBuild property [`RestoreLockedMode`](https://github.com/NuGet/Home/wiki/Enable-repeatable-package-restore-using-lock-file#extensibility) to control this behavior.

## Lock File format

We discussed multiple options such as `Yaml`, `Json` or `Xml` as format to generate this lock file and laid down some basic principles to choose one. Based on those principles, we prototyped `Yaml` and `Json` formats to compare the different properties for both formats and select the most appropriate one.

Based on these prototypes, we should go ahead with `Json` format to maintain this lock file. Below are the reasoning based on the principles we set to select the file format:

* Already have well known parser so no new dependency or writing your own parser which will also take several iterations to get that right
* Performance of `JSON` vs `YAML` isn't too bad specially while reading the file which will happen too often compare to write.
  * Writing a sample [`packages.lock.json` file](https://github.com/NuGet/Home/blob/dev/resources/RepeatableBuildLockFile/project.lock.json) takes ~35ms compare to writing the content as [`packages.lock.yaml`](https://github.com/NuGet/Home/blob/dev/resources/RepeatableBuildLockFile/project.lock.yaml) takes ~8ms
  * Reading a sample `packages.lock.json` file takes ~2.09ms compare to reading the content as `packages.lock.yaml` takes ~1.35ms
* Easy to identify what changed and easy mergable
* Json file looks more human readable and easy to understand
* Consistent with existing NuGet artifacts

## Open Items

* Finalize UI option to run restore with reevaluate restore graph even with a lock file. Also get rid of rebuild running force restore inside VS.