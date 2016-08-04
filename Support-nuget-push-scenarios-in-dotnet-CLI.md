# Problem

dotnet CLI doesn't yet have a push scenario--the only way to effect a nuget push from the command line is by using the nuget CLI.

# Who is the customer?

Package authors

# Evidence

*still amassing*

# Solution

nuget.exe push options are defined here: [NuGet Push Command Options](https://docs.nuget.org/consume/command-line-reference#push-command-options)

The proposal is to insert a command into the dotnet CLI to target nuget, and push will be a option of that command (with the potential for other options). The dotnet CLI will route all arguments to the push command, ensuring behavior consistent with nuget.exe push. Examples of the new command line are:

dotnet nuget push foo.nupkg 4003d786-cc37-4004-bfdf-c4f3e8ef9b3a

dotnet nuget push foo.nupkg 4003d786-cc37-4004-bfdf-c4f3e8ef9b3a -s http://customsource/

dotnet nuget push foo.nupkg

dotnet nuget push foo.symbols.nupkg

dotnet nuget push foo.nupkg -Timeout 360

dotnet nuget push *.nupkg