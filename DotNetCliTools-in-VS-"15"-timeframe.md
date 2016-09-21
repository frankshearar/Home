## Problem
How do DotNetCliTools change from Preview2?

## Who is the customer?
* DotNet Devs using dotnetclitools
* DotNet Devs building dotnetclitools

## Solution
### toolsref in csproj
We need to finalize on the technique here. Two ideas currently on the table.

#### DotNetCliToolsReference Technique
    <DotNetCliToolsReference Include="Microsoft.AspNet.EF.Tools" Version="1.0.0" />
    [Note .. version won’t work as an attribute until RC…waiting on Msbuild feature. Until then use version child element…]

If you wanted to list it as both a packageref and a tools ref, you would:
    <DotNetCliToolsReference Include="Microsoft.AspNet.EF.Tools" Version="1.0.0" />
    <PackageReference Include="Microsoft.AspNet.EF.Tools" />

#### PackageReference UsePackageAs Technique
    <PackageReference Include="Microsoft.AspNet.EF.Tools" Version="1.0.0">
      <UsePackageAs>DotNetCliTool</UsePackageAs>
    </PackageReference> 

If you wanted to use it as a packageref and a tools ref, you would:
    <PackageReference Include="Microsoft.AspNet.EF.Tools" Version="1.0.0">
      <UsePackageAs>DotNetCliTool</UsePackageAs>
    </PackageReference> 

### tools hive
why is the tools lock file in .nuget\tools?
consider moving to $(baseintermediatepath)
nuget restore -out ..\foo
solution level restore could do 1 restore per tool
ee - how long does starting msbuild to get a property take.

### imports
scenario - something runs on mono on windows.
relied on an API that PInvokes
want to choose a pinvoke-less implementation from nupkg.
    <PackageReference Include=”foo.bar”>
      <Version>1.2.3</Version>
      <TreatAs>Net45</TreatAs>
    </PackageReference>
    // Which means…<IfYouUseTHisPackageId,Treat it as Net45 />

### - how does cli discover tools?
[TODO]

### More open issues?
[1-4 was the set we discussed last time, not sure if there are more]
