Content v2 is immutable shared content for project.json
This spec covers NuGet support for packaging content items with build properties.

*note* if your project is using packages.config to define packages, this feature is explicitly turned off. See https://github.com/NuGet/Home/wiki/Converting-a-csproj-from-package.config-to-project.json on how to port over to project.json 

Examples of content v2:
* Images that are embedded as resources
* Source files that are compiled
* PP files that are transformed to match the project they are included in
* Directories of scripts that need to be copied to the output directory with the same folder structure.

## Nupkg
A new folder will be used in the nupkg for shared content. All items in the shared folder must have a code language folder and a TxM folder.  Within the TxM folder sub folders can also exist.

Pattern:
``/contentFiles/{codeLanguage}/{TxM}/{any?}``

Language and TxM agnostic content:
``/contentFiles/any/any/config.xml``

net45 content for all languages:
``/contentFiles/any/net45/config.xml``

CSharp specific content for net45 and up:
``/contentFiles/cs/net45/config.xml``

Empty folders can use ``_._`` to opt out of providing content for certain combinations of language and TxM:
```
/contentFiles/vb/any/code.vb
/contentFiles/cs/any/_._
```

## Nuspec
Nuspecs can contain a contentFiles section which applies additional properties to items in the contentFiles folder.

```xml
<contentFiles>
    <!-- Embed image resources -->
    <files include="any/any/images/dnf.png" buildAction="EmbeddedResource" />
    <files include="any/any/images/ui.png" buildAction="EmbeddedResource" />
    <!-- Embed all image resources under contentFiles/cs/ using a wild card -->
    <files include="cs/**/*.png" buildAction="EmbeddedResource" />
    <!-- Copy config.xml to the root of the output folder -->
    <files include="cs/uap10.0/config/config.xml" buildAction="None" copyToOutput="true" flatten="true" />
    <!-- Copy run.cmd to the output folder and keep the directory structure -->
    <!-- Include everything in the scripts folder except exe files -->
    <files include="cs/uap10.0/scripts/*" exclude="**/*.exe"  buildAction="None" copyToOutput="true" />
    <!-- All other files in shared are compiled and use the default options -->
</contentFiles>
```

The include and exclude properties on contentFiles/files elements support wildcards using the aspnet syntax.
https://github.com/aspnet/FileSystem

If multiple entries match the same file all entries will be applied. The top most entry will override the lower entries if there is a conflict for the same attribute.

The contentFiles section is optional, by default all files in the nupkg contentFiles directory will use the default attribute values shown below.

### Nuspec files attributes
|Attribute|Description|
|-------------|----------------------------------------------------|
|include|[Required attribute] Include provides either a file path or a wild card path. All matching files from the contentFiles folder will have the attributes for that files node applied. Examples: ``**/*``, ``**/*.cs``, ``any/any/myfile.txt``, ``**/net*/*``.|
|exclude|Exclude provides either a file path or a wild card path. All matching files will be excluded from the include.|
|buildAction|Build action taken by msbuild for the content items. Examples: ``None``, ``Compile``|
|copyToOutput|If True the content items will be copied to the build output folder|
|flatten|If False the content items will be copied to the build output folder using the full folder structure from the nupkg. This path will be relative to the TxM folder. Example: cs/net45/config/data.xml -> config/data.xml|

#### Attributes defaults
|Attribute|Value|
|-------------|----------------------------------------------------|
|buildAction|Compile|
|copyToOutput|False|
|flatten|False|

## Lock file
Items from the contentFiles folder in the nupkg are displayed in the lock file under contentFiles for target packages. 
### ContentFiles item properties

|Property|Type|Description|
|-------------|------|----------------------------------------------------|
|buildAction|string|Equivalent to the build action drop down shown under properties in visual studio.|
|copyToOutput|bool|If true the file will be copied to the output directory.|
|outputPath|string|Path relative to the output directory where the file will be copied. This is only given if copyToOutput is true. If flatten is true this will be the file name. If fallen is false this will be the full path relative to the TFM folder.|
|ppOutputPath|string|Output path of the pp transform. This is the pp file path without .pp, and relative to the TFM folder. This is only displayed for .pp files.|
|codeLanguage|string|This property is taken from the file path.Example values: cs, vb, any. MSBuild will select the nearest language, and select all items matching that exact value.Example: If the language is CS and both CS and any exist, only CS will be selected. MSBuild should not combine items marked with different language properties, NuGet handles this.|

### Example
```json
"SharedContentA/1.0.0": {
  "contentFiles": {
     "contentFiles/cs/uap10.0/code/util.cs": {
       "buildAction": "compile",
       "codeLanguage": "cs",
       "copyToOutput": false
      },
     "contentFiles/cs/uap10.0/code/Foo.cs.pp": {
       "buildAction": "compile",
       "codeLanguage": "cs",
       "copyToOutput": false,
       "ppOutputPath": "code/Foo.cs"
     },
     "contentFiles/cs/uap10.0/scripts/run.cmd": {
       "buildAction": "None",
       "codeLanguage": "cs",
       "copyToOutput": true,
       "outputPath": "scripts/run.cmd"
     }
    }
  }
}
```

## PP transforms
PP transforms in NuGet work by replacing all tokens with the corresponding msbuild property value. Tokens are written in the format: $word$. Transforms in the shared folder will be limited to a specific set of properties which will be supported in both msbuild and DNX. Today $rootnamespace$ accounts for 90% of tokens usage in .pp transforms on nuget.org.

### PP token parsing
A PP token is defined with $'s in the format $word$. A word is a contiguous string of word characters as defined by: https://msdn.microsoft.com/en-us/library/20bw873z.aspx#WordCharacter

The escape sequence for $ is $$. When the parser encounters $$ it will return a single $ as text. $$ is not allowed inside a token word, it is only for text outside of tokens.

When a token has been identified NuGet replaces the token string with the corresponding property value. All other text is written out to the file as-is.

The current NuGet tokenizing code is available here:
https://github.com/NuGet/NuGet.Client/blob/dev/src/NuGet.Core/NuGet.ProjectManagement/Utility/Tokenizer.cs

### Error handling
If the msbuild property for a token does not have a value NuGet throws an InvalidOperationException with the message "The replacement token '{0}' has no value." MSBuild and DNX will need to support this also by failing the build with a similar error message letting the user know which property needs to be set in order to use the package.