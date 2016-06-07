# Shared packages folder

Shared packages folders allow packages to be shared across users and machines to reduce disk space. These folders are treated as fallback folders for the primary global packages folder (``%USERPROFILE%\.nuget\packages``). They differ from package sources in that the package assets will be referenced directly and will not be copied into the user's packages folder.

The concept of a shared package folder can be thought of as a GAC for nupkgs.

### Goal
The goal of the shared packages folder is to reduce disk space. Common packages such as NetStandard.Library can exist on a network share and multiple users may reference the package assets directly from the share.

### Precedence
Package folders have an order of precedence. During restore and build the user's packages folder should be checked first for a package, then each shared packages folder in order. Once a package has been found the search stops. Search is based on the id and version of the package only. Where the package was originally downloaded from and other asset information is not considered.

The user's package folder is *always* overrides shared packages folders.

