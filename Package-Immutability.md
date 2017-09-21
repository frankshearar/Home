Status: **Incubation**

With [Package Signing](https://blog.nuget.org/20170914/NuGet-Package-Signing.html), we have two primary goals: **Package Integrity** and **Package Authenticity**

The [package signing blog post](https://blog.nuget.org/20170914/NuGet-Package-Signing.html) calls out several key design principles - this spec focuses on **Package Immutability**.

## Issue
The discussion around this spec is tracked here - Package Immutability [#5917](https://github.com/NuGet/Home/issues/5889)

## Need for Package Immutability
To guarantee the integrity of a package, the package contents must not change from the time it was authored and signed to when a developer consumes it i.e. the package contents must be immutable, this includes the `nuspec`. Editing the package metadata results in changes to the nuspec, invalidating existing signatures. Thus, editing package metadata violates the key design principle - Package Immutability.

## Solution
On NuGet.org, there are two ways you can edit the package metadata -
* verify stage of the package upload workflow
* edit package button on a published package

To adhere to the design principle of package immutability, the the ability to edit package metadata will be phased out.

For signed packages, the package cannot be edited on NuGet.org.

For unsigned packages, we have a phased approach:

### Warning banners on package metadata edit pages
These banners will call out the recommendation of now editing a package after it has been authored and point to a readme link that explains the reasoning for this recommendation.
