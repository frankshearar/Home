## There are several issues in current configuration settings.

1. NuGet setting dialog does not reflect hierarchical NuGet.Config. Currently, it’s confusing about where a package source comes from, where a package source goes to during adding new source. We should come up a new UI that makes user can modify each NuGet.Config on UI.

1. Currently there is no way to add username/password for each source on UI, user always need to use nuget.exe to add them, and also need to reopen the current solution to make the new setting take effect.

1. "clear" tag and NuGet.config priority are really confusing on Setting UI. Current setting UI doesn’t reflect which NuGet.Config contain “clear” tag and priority for each NuGet.Config. For example, user clone a solution from github and open it in VS, if there is a NuGet.config in this solution and it contains "clear" tag. Then when user open configuration setting, his global sources are missing, he will have no idea what happens unless he understands the "clear" tag thing.

We should address those issues in new configuration setting.

## On the new configuration setting, it should have following feature:

1. Dialog window for each NuGet.Config, user can add and remove package source for each NuGet.config there. 

1. Credential setting for each package source, user can add username and password for each source.

1. On each NuGet.config window, it should show information about clear tag, NuGet.config location and priority for current NuGet.config.

Discussion bug for NuGet Hierarchical Setting : https://github.com/NuGet/Home/issues/1820
