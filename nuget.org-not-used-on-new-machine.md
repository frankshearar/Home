## Symptom

NuGet restore fails on a new machine if Chocolatey (`choco`), or PowerShell's `Install-Module` was used before using NuGet or Visual Studio for the first time. It may result in an error message similar to the following:

> Package 'Newtonsoft.Json.10.0.3' is not found on source 'C:\Program Files (x86)\Microsoft SDKs\NuGetPackages\'

Notice that the only source provided is `C:\Program Files (x86)\Microsoft SDKs\NuGetPackages\`. No `nuget.org` URL was listed.

## Mitigation

Use NuGet before using Chocolatey or PowerShell Gallery. 

## Solution

There are two options. One is to delete the NuGet.Config file that Chocolatey or PowerShell created. The other option is to add https://api.nuget.org/v3/index.json as a package source to the NuGet.Config file.

### Delete the generated NuGet.Config file

In PowerShell, you can use the following command to delete your `NuGet.Config` file:

```text
rm $env:USERPROFILE\NuGet\NuGet.Config
```

Using Command Prompt

```text
del %appdata%\NuGet\NuGet.Config
```

As Chocolatey and PowerShell Gallery are typically used on Windows, rather than Linux or Mac, it's less likely that the root cause of the problem is the same, but on these platforms if you wish to delete your user-profile NuGet.Config file, you can run the following command on any terminal

```text
rm ~/.nuget/NuGet/NuGet.Config
```

### Add nuget.org as a package source

To add nuget.org as a package source in Visual Studio, open Tools->Options, NuGet Package Manager->Package Sources. Click the green +, enter nuget.org as the Name, and enter https://api.nuget.org/v3/index.json as the Source. Click the Update button, then click the Ok button.

To add nuget.org using the `dotnet` CLI (command line interface), run `dotnet nuget add source https://api.nuget.org/v3/index.json --name nuget.org`.

To add nuget.org using the NuGet CLI (nuget.exe), run `nuget.exe sources add -Name nuget.org -Source https://api.nuget.org/v3/index.json`.

If you would like to edit NuGet.Config yourself, add `<add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />` under the `<packageSources>` element.

The minimal complete file contents are:

```xml
<?xml version=""1.0"" encoding=""utf-8""?>
<configuration>
  <packageSources>
    <add key=""nuget.org"" value=""https://api.nuget.org/v3/index.json"" protocolVersion=""3"" />
  </packageSources>
</configuration>
```



## Root Cause

In 2016 a bug was fixed in NuGet where [nuget.org was not automatically as a package source when the user-profile NuGet.Config file was created](https://github.com/NuGet/NuGet.Client/commit/2cef46a2ae0298e4d6bc5dfd491d6cf98a2c7198). A few weeks later, NuGet was modified to [automatically add nuget.org into the user-profile NuGet.Config file](https://github.com/NuGet/NuGet.Client/pull/458) when the file exists, but it didn't contain any package sources. This allowed customers who used NuGet before the first bug fix to get nuget.org as a package source. It used a second tracking file to allow customers to remove the nuget.org source if they wish, without it being automatically added back again.

In early 2021, following a security researcher's blog post, guidance came out, for example [Microsoft's "3 Ways to Mitigate Risk When Using Private Package Feeds" whitepaper](https://azure.microsoft.com/en-us/resources/3-ways-to-mitigate-risk-using-private-package-feeds/), which effectively recommends that customers have a single private package source where they either copy all the packages they need, or has "upsource" functionality built into the NuGet server, so the NuGet client only uses the single package source. Therefore there were a number of customers, who never used Chocolatey or PowerShell Gallery, who discovered that they needed to remove nuget.org from their config twice.

This is because the first time NuGet is run on a machine, it generates `NuGet.Config` with nuget.org as a package source. The customer removes nuget.org from the package sources. The next time NuGet runs, it sees the package sources in the user-profile `NuGet.Config` is empty, and the tracking file does not exist, so it adds nuget.org as a package source, and writes the tracking file. Finally, the customer can remove nuget.org as a package source, and subsequent times NuGet runs, even if the package sources list is empty, it sees the tracking file and does not attempt to add nuget.org again.

Chocolatey and the version of PowerShell that is built into Windows, used old versions of the NuGet SDK, with the bug that existed before 2016 where nuget.org was not added as a package source when the file was created for the first time. PowerShell Core (PowerShell 6 and above) has since updated to a newer version of NuGet which does not have this problem. Similarly Chocolately had forked NuGet.Core, which contained the bug where nuget.org was not added. After being advised of this issue, they updated their fork to stop writing NuGet.Config to the default location, since they don't use it.

Going forward, as customers use newer versions of Chocolatey and PowerShell 7 or above, this issue should no longer occur.