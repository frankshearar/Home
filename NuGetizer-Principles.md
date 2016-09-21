The following principles guide the NuGetizer design:

**1.** It should be easy to create a package from any library project. When packaging an individual project, no additional packaging projects should be necessary.

**2.** Creating a package that contains multiple platform-optimized implementations (such as "bait and switch" packages) should be as easy as File->New Project. This is an extremely important use case and must be first class.

**3.** It should be easy to add platform-specific implementations to a cross-platform package. Developers who start out creating a simple package from a netstandard project must be able to add platform-specific variants without recreating the project.

**4.** It should not normally be necessary for a developer to specify or override which files are included in a package, but they must be able to do so.

**5.** Platform-specific targets should be able to inform and override the packaging behavior to handle files specific to that platform. The Common targets should not assume all platforms work the same.

**6.** Projects and packages should be as interchangeable as possible. Referencing a project should be equivalent to referencing the package that it would create.