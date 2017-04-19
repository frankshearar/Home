## Problem
How do DotNetCliTool references change from Preview2?

## Who is the customer?
* DotNet Devs using dotnetclitool references
* DotNet Devs building dotnetclitool references

## Solution
### toolsref in csproj
Plan of Record for Preview 5 OOB:

              <DotNetCliToolsReference Include="EF.Tools">
                    <Version>2.0.0</Version>
              </DotNetCliToolsReference>

Restore location for Tools Today:

    %userprofile%\.nuget\tools\EF.Tools\1.0.0
    %userprofile%\.nuget\tools\EF.Tools\2.0.0
              
Restore location for Tools in P5OOB:

    %userprofile%\.nuget\toolsPreview5\EF.Tools\1.0\project.assets.json
    %userprofile%\.nuget\toolsPreview5\EF.Tools\1.0\<SomeHexHashForImports>\project.assets.json

CLI Team:
    Will enhance CLI to be able to run tools that were registered/restored as above.

Plan of Record for Post-Preview 5 OOB:
There are many issues that people wanted to review/improve in this area. Let’s have the longer term discussion around 9/30 as the P5OOB fires are all out.

