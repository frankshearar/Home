Status: **Reviewing**

This specification is one part of a new experience for package signing described in  the blog post:  [NuGet Package Signing](https://blog.nuget.org/20170914/NuGet-Package-Signing.html).

### Package Signatures Master Spec List

Here you can find a list of the relevant specifications. Some of these require more work and details to be added, that we plan to do shortly – while some are further along. They are grouped by the three stages described in the blog post [NuGet Package Signing](https://blog.nuget.org/20170914/NuGet-Package-Signing.html).

The work for this feature and the discussion around the spec is tracked here: [#2577 Package Signing](https://github.com/NuGet/Home/issues/2577)

#### Stage 1. Enable package authors to sign their packages
- **Author Package Signing**: Describes the user experience for producing and consuming signed packages. 
    - https://github.com/NuGet/Home/wiki/Author-Package-Signing. 
    - Discuss [#5889](https://github.com/NuGet/Home/issues/5889)

- **NuGet.exe Sign Command**: Describes the CLI sign command in NuGet.exe to sign packages
    - https://github.com/NuGet/Home/wiki/NuGet-Sign-Command
    - Discuss [#5907](https://github.com/nuget/home/issues/5907)

- **NuGet.exe Verify Command**: Describes de CLI verify command in NuGet.exe to verify package signatures
    - https://github.com/NuGet/Home/wiki/NuGet-Verify-Command
    - Discuss [#6005](https://github.com/nuget/home/issues/6005)

- **Package Signatures Technical Details**: Contains the signature format technical details 
    - https://github.com/NuGet/Home/wiki/Package-Signatures-Technical-Details
    - Discuss [#6250](https://github.com/nuget/home/issues/6250)

- **Register package signing certificates on NuGet.org**: Contains the certificate registration flow for NuGet.org authors 
    - https://github.com/NuGet/Home/wiki/Register-package-signing-certificates
    - Discuss [#5346](https://github.com/nuget/NuGetGallery/issues/5346)


#### Stage 2. Tamper proofing entire package dependency graphs 
- **Repository Signatures**
    - https://github.com/NuGet/Home/wiki/Repository-Signatures
    - https://github.com/NuGet/Home/wiki/Repository-Signatures-and-Countersignatures-Technical-Specification
    - Discuss [#6378](https://github.com/NuGet/Home/issues/6378) 

#### Stage 3. Configurable policies to enable locked down developer environments
- [[[Spec] NuGet Package Signing Client Policy]]
- [[[Spec] NuGet Config schema changes to enable trusted signers]]