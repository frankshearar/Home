## Problem
We need to migrate xproj/project.json data to msbuild based data structures in VS "15" for .NET Core.

## Who is the customer?
.NET Core developers

## Solution
### Simple case
    <PackageReference Include="NuGet.Versioning">
        <Version>3.6.0</Version>
    </PackageReference>
    <!-- this means the same as 3.6.0 in project.json today, which is >=3.6.0, preferring the lowest -->

#### FAQ about “/3.6.0” as part of the Include:
1. [ESJ] One set of scenarios that might be interesting is giving targets the ability to examine/control the versioning of PackageReference items.  That’s straightforward if Version is metadata, less so if it is part of a string.  I agree though that brevity is a higher priority here.  Perhaps you have a target that runs and produces an intermediate that converts these to Items + meta-data.
[YG] 
Targets controlling the versions of packagerefs would become a chicken and egg thing eventually, and will get in the way of how the UI works. But we could still probably support it with a library that can decompose/recompose. Alternatively we can have a “brevity” format and an expanded format. I’m not in love with the latter suggestion because it adds more edge cases and confusion.

### Conditioned on a TFM
`<PackageReference Include="NuGet.Versioning" Condition = "'$(TargetFramework)' == 'netstandard10'>
  <Version>3.6.0</Version>
</PackageReference>`

### Versioning Flexibility
Floating Version
<PackageReference Include="NuGet.Versioning/3.6.*" />
<PackageReference Include="NuGet.Versioning/3.6.0-beta*" />

                Issues with “*” inside of an Include:
a)	Msbuild currently will treat this “*” as a wildcard. We are seeing if there is a way for msbuild to support this syntax (perhaps https://github.com/Microsoft/msbuild/issues/982), or perhaps a “Version” attribute. If neither of those are possible, we’ll likely have to go with:
<PackageReference Include="NuGet.Versioning">
                <Version>3.6.*</Version>
</PackageReference>

We’d like to start enabling double asterisks
<PackageReference Include="NuGet.Versioning/3.6.*-beta*" />

Explicit Floating Version Ranges
<PackageReference Include="NuGet.Versioning/[3.6.0-3.7.0)" />

Hard Version
<PackageReference Include="NuGet.Versioning/[3.6.0]" />

Strict Mode
<PackageReferenceMode>strict|normal|permissive</PackageReferenceMode>
•	Strict will convert “3.6.0” to “[3.6.0]” (on top level and dependencies – across the entire restore graph)
•	Normal will warn on upgrades
•	Permissive will not warn
Current default is permissive. New default will be normal.

Asset Inclusion
Include/Exclude/Private Assets
<!-- below are the default values for these 3 settings, consistent with project.json today 
<PackageReference Include="NuGet.Versioning/3.6.0">
                <IncludeAssets>all</IncludeAssets>
                <ExcludeAssets>none</ExcludeAssets>
                <PrivateAssets>contentfiles,analyzers,build</PrivateAssets>
</PackageReference>

                IncludeAssets – These assets should be consumed
                ExcludeAssets – The opposite of include
                PrivateAssets – Consume but do not flow to the next project
                [Note: PrivateAssets is a new term for XProj/Project.json’s SuppressParent – we are open to other name suggestions, but we believe this is an improvement.]


All three of these can include any of the following values:
•	Compile – are the contents of the lib folder available to compile against
•	Runtime – are the contents of the runtime folder distributed
•	ContentFiles – are the contents of the contentfiles folder used
•	Build – do the props/targets in the build folder get used
•	Analyzers – do the analyzers get used

Or, instead:
•	None – none of those things get used
•	All – all of those things get used.

Type=Platform
We’ll be talking with CLI folks about making the need for this to go away

Type=Build
If we are able to get rid of the need for type=platform, we’d like to make type=build as simple as:
<PackageReference Include="NuGet.Versioning/3.6.0">
                <PrivateAssets>all</PrivateAssets>
</PackageReference>            
Coming soon
Potential addition of package control:
•	Hash
•	PublicKey
•	Signature
•	FeedName
•	FeedUrl

PackageReference and ProjectReference Duality
There should be a transformation between ProjectReference and PackageReference that keeps the result of the build identical.
Visual Studio, and other tools, could provide a feature to switch between those 2 modes for a project/package.

As such:
1)	metadata from PackageReferences may be needed to be respected on ProjectReference. IncludeAssets, ExcludeAssets and PrivateAssets will also be specifiable ProjectReference.
2)	FrameworkReferences, ProjectReferences or PackageReferences should flow transitively.
