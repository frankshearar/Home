# Problem

dotnet CLI doesn't yet have a push scenario--the only way to effect a nuget push from the command line is by using the nuget CLI.

# Who is the customer?

Package authors

# Evidence

*still amassing*

# Solution

nuget.exe push options are defined here: [NuGet Push Command Options](https://docs.nuget.org/consume/command-line-reference#push-command-options)

The proposal is to insert a command into the dotnet CLI to target nuget, and push will be an option of that command (with the potential for other options). The dotnet CLI will route all arguments to the push command, ensuring behavior consistent with nuget.exe push. 

### Usage
    dotnet nuget push <package path> [API key] [options]

Specify the path to the package and your API key to push the package to the server.

### Options
<table>
    <tr>
        <td>apikey</td>
        <td>The API key for the server.</td>
    </tr>
    <tr>
        <td>configfile</td>
        <td>(v<em>2.5</em>) Specifies the user specific configuration 
        file. If omitted, nuget.config is used 
        instead. </td>
    </tr>
    <tr>
        <td>help</td>
        <td>Displays help information for the push command.</td>
    </tr>
    <tr>
        <td>noninteractive</td>
        <td>Do not prompt for user input or confirmations.</td>
    </tr>
    <tr>
        <td>source</td>
        <td>Specifies the server URL. This is a mandatory parameter unless the NuGet.config file specifies a 
        DefaultPushSource value.
        <br />
        if dotnet nuget push identifies a UNC/folder source, it will copy the file to the source.
        </td>
    </tr>
    <tr>
        <td>timeout</td>
        <td>Specifies the timeout for pushing to a server in seconds. 
        Defaults to 300 seconds (5 minutes).</td>
    </tr>
    <tr>
        <td>verbosity</td>
        <td>Specifies the amount of details displayed in the output: normal, 
        quiet, (v<em>2.5</em>) detailed.</td>
    </tr>
</table>

### Examples

    -Source is a mandatory parameter unless DefaultPushSource config value is set in the NuGet config file.
    dotnet nuget push foo.nupkg 4003d786-cc37-4004-bfdf-c4f3e8ef9b3a -Source https://www.nuget.org/api/v2/package
    
    dotnet nuget push foo.nupkg 4003d786-cc37-4004-bfdf-c4f3e8ef9b3a -Source https://www.nuget.org/api/v2/package

    dotnet nuget push foo.nupkg 4003d786-cc37-4004-bfdf-c4f3e8ef9b3a

    dotnet nuget push foo.nupkg 4003d786-cc37-4004-bfdf-c4f3e8ef9b3a -s http://customsource/

    dotnet nuget push foo.nupkg

    dotnet nuget push foo.symbols.nupkg

    dotnet nuget push foo.nupkg -Timeout 360

    dotnet nuget push *.nupkg

    dotnet nuget push will support a UNC/Folder source:
    dotnet nuget push -source \\mycompany\repo\ mypackage.1.0.0.nupkg
