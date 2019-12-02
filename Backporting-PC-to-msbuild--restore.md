[@jeffkl](https://github.com/jeffkl) was kind enough to put together [this PR](https://github.com/NuGet/NuGet.Client/pull/2917) that backports PC-based installation to msbuild.exe, in a limited way. This is a great starting point and the limited scope means we're more comfortable doing it! This document is a discussion about requirements and next steps towards integrating this code and taking it over the finish line.

## Rationale

We've long had the stance that projects using PC should be moved over to PR, because of how much better an experience it is. We also have an automatic migrator to help folks do this. Unfortunately, two main project types do not support PR yet: Service Fabric (.sfproj), and Visual C++ (.vcxproj), and there's a number of large projects, both internal and external, who are already using msbuild with PC.

While we would prefer these project types add support for PR, it's mostly out of our hands and in the meantime, users need to be able to use tools like msbuild. 

## Goals

* Provide enough feature parity that .sfproj and .vcxproj tools can be restored using msbuild, until such a time when they can actually migrate to PR.
* Communicate PC's status and our intentions with users when things that seem like they should work don't, or if certain features of PC restores happen to be missing.

## Proposal

* Take over @jeffkl's PR and take it over the finish line ourselves, modulo maybe some customer testing help
* Add end-to-end test coverage
* Continue to only support .NET Framework and not .NET Core
* Only work by default for whatever VS supports
* Make the feature opt-in.
* Allow PackageSaveMode from config, not from command line flag
* Don't port DirectDownload over

### Later

* Warn as follows:
  * when not opted-in
    * if has PC files that are upgradable
      * tell them, we didn't restore it and to consider moving to PR (aka.ms/pleaseUpgradePeople)
    * if has PC files that are not upgradable
      * tell them, we have a feature they can opt into (aka.ms/howToRestorePCFiles)

  * when opted-in
    * if has PC files that are upgradable
      * tell them to consider moving to PR (aka.ms/pleaseUpgradePeople)
    * if has PC files that are not upgradable
      * tell them we restored it (via normal restore output)
  * Make warnings disableable through `nowarn` and possibly a flag
  * (even later) Make dotnet and dotnet msbuild warn for all types, if there's a P.C. proj
  * (even later) With 17.0/6.0, start warning in nuget.exe and VS as well

## See Also

* [The original pull request](https://github.com/NuGet/NuGet.Client/pull/2917)
* [@jeffkl on Github](https://github.com/jeffkl)
