# Lock file design meeting notes

In this meeting we covered lock file design, package install and update scenarios for locked projects, and discussed how the current project locking scenarios could be improved by introducing a snapshot file to store only the locked package versions.

Please use the design meeting [discussion issue](https://github.com/NuGet/Home/issues/1233) to provide feedback, ask questions, etc.

Meeting questions with answers noted below:
* When is a lock file with IsLocked=true allowed to change
  * If dependencies have changed
  * If project.json has changed
* Are there any scenarios where NuGet should change IsLocked to false after an operation
  * No scenarios came up where the lock file should be marked as IsLocked=false
* Installing a new package to a project that has a locked lock file
  * This is allowed. The lock file should stay locked.
* Installing into a project that has IsLocked=false, what is the result on the parent projects that have IsLocked=true
  * Mixing locked and unlocked projects should fail. How this behaves exactly is still an open question.
* For projects with IsLocked=true that contain floating versions, what is the expectation when other packages in the project are changed and the lock file needs to be updated
  * This will float all other packages again. Users should work around this by adding the package to project.json if they do not want it to float.
* Corrupted lock files, is it ever safe to ignore an unreadable lock file and create new one if the corrupted file may have been locked
  * This would be easier with a snapshot file

Lock files are the view for a single project. They contain transitive references, but purely from the view of that project.

Restoring on build as it happens today is wrong.

A tool was originally planned to make a project.json file ready for release which locks all the versions in the project.json file. (Make release)

A feature could be added to stop automatic floating. Ex: project.json has 1.0.0  if only 1.0.1 exists due to the source changing and no packages are asking for 1.0.1 the restore would fail. This is not currently implemented today.

A possible option for fixing the lock file issue today is adding a 3rd file which contains the actual versions of all packages used. There are open questions on validation between the snapshot, lock file, and project.json file.
* This fixes the lock file size issue
* Source controlled lock files
* The snapshot file would only be generated when the user locks the project, and it would then be checked into source control
* Users who wanted to float versions would not have a snapshot file. Without the snapshot the floating versions would be selected each time.
* The lock file would become more of a build artifact could be moved to the .nuget folder or another hidden folder
* Snapshot files would encounter merge conflicts like packages.config today. For scenarios where this is common the user would want to float versions and not lock the package versions.

NuGet lock to generate the snapshot

NuGet restore unlock to return to floating versions


