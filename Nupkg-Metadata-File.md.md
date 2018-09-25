# Package Nupkg Metadata file

* Status: **Incubation**
* Author(s): [Ashish Jain](https://github.com/jainaashish)

## Issue

[7283](https://github.com/NuGet/Home/issues/7283) - NuGet package original hash
 
 ## Problem
 Today while installing a NuGet package, we write down the nupkg file hash in a separate file `<packageId>.<packageVersion>.nupkg.sha512` which is then used to understand if the package available on disk so that we don't extract it again. Now, this means for signed packages this hash will be different than the original hash of the same package when it was not signed or repro signed. 
 
 This will create major issues when people opt into [Project lock file feature](https://github.com/NuGet/Home/wiki/Repeatable-build-using-lock-file-implementation) since as part of that feature, we'll start doing package hash validation as part of restore. And when customer move from unsigned to signed package or author signed to author + repo signed package, then this hash validation will fail which will also fail the restore.
 
 ## Solution
 
 After discussing multiple approaches and their pros/ cons (which are listed in the associated issue), we decided to have a new metadata file which will always have original SHA512 hash of the package content excluding signature metadata. So this hash will always be consistent across unsigned or signed (author or repo) package as long as package content is not tempered or modified. We'll still continue to generate the existing hash file with the same content (hash of the nupkg file) for backward clients compatibility.
 
 Here are the details:
 
 * In order to figure out if package exists in the global packages folder, we'll now use the new hash file (`<packageId>.<packageVersion>.metadata.json`) instead of existing hash file. So anytime the new hash file doesn't exists in the global packages folder, then
   * if old hash file exists, then we'll generate the new hash from existing nupkg and write out the new hash file.
   * Else, we'll extract it from the configured feeds and create both old as well as new hash files at the current location (which is the package root).
   
 * We'll continue to check for existing hash file in restore noop check in order to figure out if package exists on the disk or not along with adding a OR check for new hash file as well. So it allows the flexibility to remove existing hash file in future without impacting old clients.
 
 * We've already communicated this change to the SDK team and they will refresh their fallback folder to have the new hash file as well. Similarly we'll communicate this change to other partners who also use fallback folders.
 
 * UNC based V3 folder feeds will continue to have `*.nupkg`, `*.nuspec`, `*.nupkg.sha512` files since we never actually `*.nupkg.sha512` from there to really consume as part of restore. This also means, we won't change any behavior for `nuget.exe add` or `nuget.exe init` commands which helps customer build V3 based UNC file feeds.
 
 * NuGet `Pack` command also supports an argument `InstallPackageToOutputPath` which also allows to create V3 based UNC file feed while generating `nupkg`. So with this option, pack also generates `nuspec` file and `.nupkg.sha512` file. So there won't be any change to this behavior since it doesn't impact anything for NuGet while consuming those packages.
 
 ## Metadata file format
 
 Finally our new file will be called `.nupkg.metadata` and will be of json format. Unlike existing hash file, now we're trying to be little more generic to have this new file so that we can also extend it in future to add more metadata, if needed. 
 
 As of now, there will be only two properties in this new file:
 * Version - version of the metadata file
 * ContentHash - SHA512 hash of the package content excluding signature file.
 
 Even going forward, when we'll need to add more metadata in this file, we'll try and keep it to key-value pair unless we really need to introduce complex object structure.
 
Sample new hash file:
```  
 {
  "version": 1,
  "contentHash": "NhfNp80eWq5ms7fMrjuRqpwhL1H56IVzXF9d+OIDcEfQ92m1DyE0c+ufUE1ogB09+sYLd58IO4eJ8jyn7AifbA=="
 }
``` 

## Meeting notes from 9/14/2018:

Issue: What should be the new file name?
- packageid.version.nupkg.metadata
- .metadata
- .nupkg.metadata - selected
- .package.metadata
- .nuget.metadata

File location – It will be under package root alongside existing *.sha512 file.

Issue: How to generate this new file?
- redownload at extraction if new hash file doesn’t exists
- try download and fallback to existing nupkg if package is not available at any feed.
- use existing nupkg n generate new file --- Selected

Issue: How should it impact restore noop?
- noop should check for new file and fail if not there - NK
- noop should check for new file and generate if not there - AJ, DT
- noop check for new file and fail if not there. Then a new pre-restore check for old file and generate a new file. Else, it goes for restore. - PT
- noop continue to check for old file - AA
- noop check with old file OR new file -- Selected


Issue: Sha256 vs Sha512 for new hash file?
- we recalculate hash 512 at extraction - Selected
- we add both 256, 512 hash at creation of nupkg (sign)

Issue: should we include hash algo in new file?
No, it will be taken care with Versioning of the file itself.

Also, change hash property to ContentHash in new hash file
