
# Title
Support locals command in dotnet CLI.

## Problem
Currently we do not have a cross platform implementation of nuget locals command.

## Who is the customer?
nuget cli users currently do not have a way to clear nuget cache using dotnet cli.

## Evidence
Working on it.

## Solution
Add a dotnet locals command to the dotnet cli. I am currently looking into what behavior will be supported. For now assuming that all the behavior stays the same as before, except there will be no option for clearing package-cache, since it is no longer used. 

I will update this doc as I investigate further.

## Usage 
dotnet nuget locals [options]

Locals Command Options - 

<table>
    <tr>
        <td>Clear</td>
        <td>Clear the resources in the specified cache location</td>
    </tr>
    <tr>
        <td>List</td>
        <td>List the selected local resources or cache locations</td> 
    </tr>
    <tr>
        <td>Help</td>
        <td>help</td>
    </tr>
    <tr>
        <td>Verbosity</td>
        <td>Display the amount of details in the output: normal, quiet, detailed.</td>
    </tr>
</table>

## Example
dotnet nuget locals \<all | http-cache | global-packages\> -clear