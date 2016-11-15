## Issue
https://github.com/NuGet/Home/issues/3751

## Problem
Currently, there is no way of adding, update and removing package references in dotnet cli

## Who is the customer?
Any one using dotnet cli.

## Evidence
The request comes from dotnet cli team that has ihnitiated the process of adding a `dotnet ref add|del|list <id>` command at https://github.com/dotnet/cli/issues/4521

## Solution
We should add support for handling `dotnet ref add|del|list <id>` where the type of reference is package reference.

* The Flow would be `dotnet ref add|del|list <id>` would invoke dotnet cli, which would detect the type of reference.
* If the type of reference is `PackageReference` then they would pass the information to a NuGet api.
* We will then use msbuild(?) api to perform the task.

## Open Questions 

* **PreRelease Switch** - We need a pre-release switch/flag from `dotnet ref` command. This would work the same way as the pre-release check box in the VS UI. <br>
e.g. - `dotnet ref add Newtonsoft.Json -p|--prerelease`

* **Update sub verb** - From NuGet's perspective an update ref sub-command makes sense. <br>
e.g. - `dotnet ref update Newtonsoft.Json -v|--version <version-string>`

* **Empty Version String** - The behavior when user does not specify an exact version should be consistent with other nuget operations.<br>
e.g. - `dotnet ref update Newtonsoft.Json ` -> Should be same as ->`dotnet ref update Newtonsoft.Json -v|--version '*'`

* **Multi-Targeting** - We should have support for multi targeting. <br>

<table>
    <tr>
        <td>Sr. No.</td>
        <td>Project TFM</td>
        <td>Package TFM</td>
        <td>Sample Reference</td>
    </tr>
    <tr>
        <td>1</td>
        <td>A and B</td>
        <td>A and B</td>
        <td><PackageReference ></td>
    </tr>
    <tr>
        <td>2</td>
        <td>A and B</td>
        <td>Only A</td>
        <td><PackageReference ></td>
    </tr>
</table>