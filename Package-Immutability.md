Status: **Reviewing**

With [Package Signing](https://blog.nuget.org/20170914/NuGet-Package-Signing.html), we have two primary goals: **Package Integrity** and **Package Authenticity**

The [package signing blog post](https://blog.nuget.org/20170914/NuGet-Package-Signing.html) calls out several key design principles - this spec focuses on **Package Immutability**.

## Issue
The discussion around this spec is tracked here - Package Immutability [#5917](https://github.com/NuGet/Home/issues/5917)

## Need for Package Immutability
To guarantee the integrity of a package, the package contents must not change from the time it was authored and signed to when a developer consumes it i.e. the package contents must be immutable, this includes the `nuspec`. Editing the package metadata results in changes to the `nuspec`, invalidating existing signatures. Thus, editing package metadata violates the key design principle - Package Immutability.

## Solution
On NuGet.org, there are two ways you can edit the package metadata -
* _Verify_ stage of the package upload workflow
* _Edit package_ button on a published package

To adhere to the design principle of package immutability, the the ability to edit package metadata will be phased out.

### Warning banners on package metadata edit pages
The users will be able to edit package metadata but will see a banner which calls out our recommendation of not editing a package after it has been authored and point to a _Read More_ link that explains the reasoning for this recommendation.

#### Verify step during package upload
<img src="https://github.com/NuGet/Home/blob/dev/resources/PackageImmutability/01.PNG" width="800" border="1"/>

#### Package edit page for published packages
<img src="https://github.com/NuGet/Home/blob/dev/resources/PackageImmutability/02.PNG" width="800" border="1"/>

#### Read More
**Q.** Why do you recommend uploading a new package for making changes to package metadata?

**A.** NuGet will be implementing package signing.  A design principle of package signing is that signed package content must be immutable, which includes the nuspec. Editing the package metadata results in changes to the nuspec, invalidating existing signatures.  We recommend modifying existing workflows to not require editing the package metadata after the package has been created.

### Read-only verify package step, and documentation only edit option for published packages
* Once the package signing feature goes live, the _Verify_ package step of the package upload workflow on NuGet.org will be made read-only. The page is merely to validate the information is accurate. If not, the user must cancel the upload operation, make the edits in the nuspec, and upload the package created using the updated nuspec.
* For published packages, the _edit_ button will only allow editing the Documentation for the package, other fields will be removed and will no longer be available for editing from this page. 

#### Verify step during package upload
<img src="https://github.com/NuGet/Home/blob/dev/resources/PackageImmutability/03.PNG" width="800" border="1"/>

Note - Users will still be able to choose the package visibility option at the verify step of the upload.

#### Package edit page for published packages
<img src="https://github.com/NuGet/Home/blob/dev/resources/PackageImmutability/05.PNG" width="800" border="1"/>

#### Read More
**Q.** Why do you require uploading a new package for making changes to package metadata?

**A** NuGet requires all packages to be signed.  A design principle of package signing is that signed package content must be immutable, which includes the nuspec. Editing the package metadata results in changes to the nuspec, invalidating existing signatures.  We recommend modifying existing workflows to not require editing the package metadata after the package has been created.

