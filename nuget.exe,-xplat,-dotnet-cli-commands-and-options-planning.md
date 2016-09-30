# Commands and options

# Delete
## NuGet.exe delete
*-Source
-NoPrompt
 - same as non-interactive?
-ApiKey
-Help
-Verbosity
-NonInteractive
-ConfigFile
-ForceEnglishOutput

## NuGet.CommandLine.XPlat.dll delete
-s/--source
--non-interactive
    add abbreviated option 
-k/--api-key
--help
--force-english-output 
    remove this from our commands for consistency in dotnet CLI
--config-file
    support this

(verbosity is covered via dotnet cli)

## dotnet nuget delete
As per NuGet.CommandLine.XPlat.dll delete

# Pack
## NuGet.exe pack
-OutputDirectory
-BasePath
-Version
-Suffix
-Exclude +
-Symbols
-Tool
-Build
-NoDefaultExcludes
-NoPackageAnalysis
-ExcludeEmptyDirectories
-IncludeReferencedProjects
     going away - no need to propagate to xplat
-Properties +
-MinClientVersion
-MSBuildVersion
    only a nuget.exe scenario
-Help
-Verbosity
-NonInteractive
    redundant--don't propagate to xplat
-ForceEnglishOutput

## NuGet.CommandLine.XPlat.dll pack
-b/--base-path
--build
--exclude
-e/--exclude-empty-directories
--min-client-version
--no-default-excludes
--no-package-analysis
-o/--output-directory
-p/--properties
--serviceable
    move this to metadata
--suffix
-s/-symbols
--verbosity
     (verbosity is also? covered via dotnet cli)
     --verbosity is special in pack and restore. We need to standardize.
-v/--version
--help
--force-english-output
    remove this from our commands for consistency in dotnet CLI
--tool
    more design work to do
--analyzers
    add?

## dotnet pack
-o/--output
--no-build
    if build is customized, we don't want to mess with it
--build-base-path
    don't use this
-c/--configuration
   follow CLI's call on this
--version-suffix
   change to --suffix
-s/--serviceable
  move to metadata

# Push
## NuGet.exe push
-Source
-ApiKey
-Timeout
-DisableBuffering
-NoSymbols
-Help
-Verbosity
-NonInteractive
-ConfigFile
-ForceEnglishOutput

## NuGet.CommandLine.XPlat.dll push
-s/--source
-ss/--symbols-source
    remove -ss
-t/--timeout
-k/--api-key
-sk/--symbol-api-key
    remove -sk
-d/--disable-buffering
-n/--no-symbols
--help
--force-english-output
--config-file
   need to support this
We may need to light up credential providers and --non-interactive here 
(verbosity is covered via dotnet cli)

## dotnet push
As per NuGet.CommandLine.XPlat.dll push

# Restore
## NuGet.exe restore
-RequireConsent
   research: let's make this less troublesome
-Project2ProjectTimeOut
   less relevant now. Don't propagate to xplat.
-PackagesDirectory
-SolutionDirectory
-MSBuildVersion
-Source +
-FallbackSource +
-NoCache
-DisableParallelProcessing
-PackageSaveMode
    more design work, may need to move to xplat when done
-Help
-Verbosity
-NonInteractive
   move to xplat
-ConfigFile
-ForceEnglishOutput

## NuGet.CommandLine.XPlat.dll restore
-s/--source
--packages
--disable-parallel
-f/-fallbacksource
--configfile
--no-cache
--infer-runtimes
     going away
--ignore-failed-sources
--legacy-packages-directory
--help
--force-english-output
    remove this from our commands for consistency in dotnet CLI
--non-interactive
    add here

(verbosity is covered via dotnet cli)



 




