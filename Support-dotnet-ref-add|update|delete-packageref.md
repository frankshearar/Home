## Issue
https://github.com/NuGet/Home/issues/3751

## Problem
Currently, there is no way of adding, update and removing package references in dotnet cli

## Who is the customer?
Any one using dotnet cli.

## Evidence
The request comes from dotnet cli team that has initiated the process of adding a `dotnet ref add|del|list <id>` command at https://github.com/dotnet/cli/issues/4521

## Solution
We should add support for handling `dotnet add|update|del|list pkg <id>` where the type of reference is package reference.

* The Flow would be `dotnet add|update|del|list pkg <id>` would invoke dotnet cli, which would pass the information to a NuGet api.
* We will then use msbuild.construction api to perform the task.

## Change Requests

* **PreRelease Switch** - We need a pre-release switch/flag from `dotnet ref` command. This would work the same way as the pre-release check box in the VS UI. <br>
e.g. - `dotnet add ref pkg Newtonsoft.Json -p|--prerelease`

* **Update sub verb** - From NuGet's perspective an update ref sub-command makes sense. <br>
e.g. - `dotnet update ref pkg Newtonsoft.Json -v|--version <version-string>`

* **Empty Version String** - The behavior when user does not specify an exact version should be consistent with other nuget operations.<br>
e.g. - `dotnet update ref pkg Newtonsoft.Json ` -> Should be same as ->`dotnet ref update Newtonsoft.Json -v|--version '*'`

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
		<td>netcoreapp1.0 and net46</td>
		<td>netcoreapp1.0 and net46</td>
		<td>
<pre>
  &lt;ItemGroup&gt;
    &lt;PackageReference Include="NewtonSoft.Json"&gt;
      &lt;version&gt;9.0.1&lt;/version&gt;
    &lt;/PackageReference&gt;
  &lt;/ItemGroup&gt;
</pre>
		</td>
	</tr>
	<tr  bgcolor="#FFFFFF">
		<td>2</td>
		<td>netcoreapp1.0 and net46</td>
		<td>netcoreapp1.0</td>
                <td>
<pre>
&lt;ItemGroup Condition="'$(TargetFramework)' == 'netcoreapp1.0' "&gt;
	&lt;PackageReference Include="NewtonSoft.Json"&gt;
		&lt;version&gt;9.0.1&lt;/version&gt;
	&lt;/PackageReference&gt;
&lt;/ItemGroup&gt;
</pre>
		</td>
	</tr>
	</tr>
	<tr  bgcolor="#FFFFFF">
		<td>3</td>
		<td>netcoreapp1.0</td>
		<td>netcoreapp1.0 and net46</td>
		<td>
<pre>
  &lt;ItemGroup&gt;
    &lt;PackageReference Include="NewtonSoft.Json"&gt;
      &lt;version&gt;9.0.1&lt;/version&gt;
    &lt;/PackageReference&gt;
  &lt;/ItemGroup&gt;
</pre>
		</td>
	</tr>
</table>

## Open Questions

* **Which API?** - Microsoft.Build.Construction

* **Kick off restore after add/update ref?** - In case a package ref is added or updated, VS kicks off a restore. Current answer is No.

* **Include private/public Assets?** - This is not done at any platform i.e. nuget.exe/VS etc. Current answer is No.