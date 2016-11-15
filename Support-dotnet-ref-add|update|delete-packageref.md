## Issue
https://github.com/NuGet/Home/issues/3751

## Problem
Currently, there is no way of adding, update and removing package references in dotnet cli

## Who is the customer?
Any one using dotnet cli.

## Evidence
The request comes from dotnet cli team that has initiated the process of adding a `dotnet ref add|del|list <id>` command at https://github.com/dotnet/cli/issues/4521

## Solution
We should add support for handling `dotnet ref add|del|list <id>` where the type of reference is package reference.

* The Flow would be `dotnet ref add|del|list <id>` would invoke dotnet cli, which would detect the type of reference, if it is not explicitly mentioned.
* If the type of reference is `PackageReference` then they would pass the information to a NuGet api.
* We will then use msbuild(?) api to perform the task.

## Change Requests

* **PreRelease Switch** - We need a pre-release switch/flag from `dotnet ref` command. This would work the same way as the pre-release check box in the VS UI. <br>
e.g. - `dotnet ref add Newtonsoft.Json -p|--prerelease`

* **Update sub verb** - From NuGet's perspective an update ref sub-command makes sense. <br>
e.g. - `dotnet ref update Newtonsoft.Json -v|--version <version-string>`

* **Empty Version String** - The behavior when user does not specify an exact version should be consistent with other nuget operations.<br>
e.g. - `dotnet ref update Newtonsoft.Json ` -> Should be same as ->`dotnet ref update Newtonsoft.Json -v|--version '*'`

* **Multi-Targeting** - We should have support for multi targeting. <br>

<table>
	<tr  bgcolor="#FFFFF0">
		<td>Sr. No.</td>
		<td>Project TFM</td>
		<td>Package TFM</td>
		<td>Sample Reference</td>
	</tr>
	<tr  bgcolor="#FFFFFF">
		<td>1</td>
		<td>A and B</td>
		<td>A and B</td>
		<td>
<pre>
	<code>
		&lt;ItemGroup&gt;
			&lt;PackageReference Include="Newtonsoft.Json"&gt;
				&lt;Version&gt;9.0.1&lt;/Version&gt;
			&lt;/PackageReference&gt;
		&lt;/ItemGroup&gt;
	</code>
</pre>
		</td>
	</tr>
	<tr  bgcolor="#FFFFFF">
		<td>2</td>
		<td>A and B</td>
		<td>Only A</td>
                <td>
<pre>
	<code>
		&lt;ItemGroup&gt;
			&lt;PackageReference Include="Newtonsoft.Json"&gt;
				&lt;Version&gt;9.0.1&lt;/Version&gt;
				&lt;TargetFramework&gt;"A"&lt;/TargetFramework&gt;
			&lt;/PackageReference&gt;
		&lt;/ItemGroup&gt;
	</code>
</pre>
		</td>
	</tr>
</table>

## Open Questions

* **Which API?** - Current information indicates that the dotnet/cli team plans on using the MSBuild API's that were used in Migration tool. 

* **Kick off restore after add/update ref?** - In case a package ref is added or updated, VS kicks off a restore.

* **Include private/public Assets?** - This is not done at any platform i.e. nuget.exe/VS etc.