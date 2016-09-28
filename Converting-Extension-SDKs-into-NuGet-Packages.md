## Issue
Feedback on the spec can be given on the following issue - https://github.com/NuGet/Home/issues/3509

## Problem
Currently ISVs and First Party Library (Framework packages) end up shipping Extension SDKs to support a variety of UWP scenarios. The reasons are both historical and in some cases NuGet does not have the complete feature set that Extension SDKs have.

Extension SDKs are inherently not very friendly to developers since they require to be installed on build/dev machines in-order for solutions to compile and does not support the seamless package management workflow that is built into NuGet.

## Who is the customer?
First Party Framework SDK authors and ISVs.

## Evidence
We have 3 primary customers for this. .NET Native, Store Services and other partners.

## Solution
There are 3 major features that we need to support to in-order to provide a seamless transition for Extension SDK authors to use NuGet.

### Appx Packages
Framework SDKs require registration of Appx packages during deployment. This can be accomplished using currently available NuGet semantics.

Appx files need to be placed in the following relative location of the NuGet package

    \tools\*

Specifically they should be tied to the architecture of the Appx package. E.g

    \tools\x86\chk\MyTestpackage.appx
    \tools\x86\ret\MyTestpackage.appx
    \tools\x64\chk\MyTestpackage.appx
    \tools\x64\ret\MyTestpackage.appx
    \tools\ARM\chk\MyTestpackage.appx
    \tools\ARM\ret\MyTestpackage.appx
    \tools\ARM64\chk\MyTestpackage.appx
    \tools\ARM64\ret\MyTestpackage.appx

The UWP tools are able to parse these paths from the lock file and correspondingly register them during deployment. This is a feature that is unique to first party packages since only they can be store serviced.

### Extension SDK Manifests

[Extension SDK manifests](https://msdn.microsoft.com/en-us/library/hh768146.aspx) are used to define the metadata around Extension SDKs. 

NuGet semantics inherently support the vast majority of the metadata in Extension SDK manifests. Therefore customers can choose to either drop in their existing manifest and we will ignore the duplicate fields in favor of the metadata specified in NuGet or they can trim it accordingly.

A sample full Extension SDK Manifest is given below

    <FileList
      DisplayName = “My SDK”
      ProductFamilyName = “My SDKs”
      TargetFramework = “.NETCore, version=v4.5.1; .NETFramework, version=v4.5.1”
      MinVSVersion = “14.0”
      MaxPlatformVersion = "8.1"
      AppliesTo = "WindowsAppContainer + WindowsXAML"
      SupportPrefer32Bit = “True”
      SupportedArchitectures = “x86;x64;ARM”
      SupportsMultipleVersions = “Error”
      CopyRedistToSubDirectory = “.”
      DependsOn = “SDKB, version=2.0”
      MoreInfo = “http://msdn.microsoft.com/MySDK”>
      <File Reference = “MySDK.Sprint.winmd” Implementation = “XNASprintImpl.dll”>
      <Registration Type = “Flipper” Implementation = “XNASprintFlipperImpl.dll” />
      <Registration Type = “Flexer” Implementation = “XNASprintFlexerImpl.dll” />
      <ToolboxItems VSCategory = “Graph”>
      <ToolboxItems/>
      <ToolboxItems BlendCategory = “Controls/sample/Graph”>
      <ToolboxItems/>
      </File>
    </FileList>


The following manifest snippet shows only the set of properties that are recognized by the tooling in Visual Studio when the manifest is read from a NuGet package.

    <FileList>
      MinVSVersion = “14.0”>
      <File Reference = “MySDK.Sprint.winmd” Implementation = “XNASprintImpl.dll”>
      <ToolboxItems VSCategory = “Graph”>
      <ToolboxItems/>
      <ToolboxItems BlendCategory = “Controls/sample/Graph”>
      <ToolboxItems/>
      </File>
    </FileList>

Extension SDK manifest needs to be placed in the root of the contentFiles directory and is optional. Manifest is primarily required to specify metadata used by the designer tools in Visual Studio and if you are not shipping controls then it is not required.

    \contentFiles\SDKManifest.xml

The semantics of the options in the trimmed manifest are spec'ed out in the original design of the SDK manifest available [here](https://msdn.microsoft.com/en-us/library/hh768146.aspx). 

### Supporting Platform Versions in UAP NuGet Packages

UAP packages have a **TargetPlatformVersion(TPV)** and **TargetPlatformMinVersion(TPM)** that defines the upper and lower bounds of the OS version where the app can be installed into. TPV further specifies the version of the SDK that the app is compiled against. 

When a NuGet package author creates a UAP library, they need to keep in mind these properties when designing and coding their libraries. Using API's outside of the bounds of the platform versions defined in the app will either cause the build to fail or the app to fail at runtime (If due dilligence is not taken into account while using adaptive APIs).

Some examples of possible combinations of TPV and TPM are given below. Abbreviations are used instead of build numbers for brevity.

| TPM | TPV |
|-----|-----|
| TH  | TH  |
| TH  | TH2 |
| TH  | RS1 |
| TH2 | TH2 |
| TH2 | RS1 |
| RS1 | RS1 |


Currently we only support the vanilla **uap** or the expanded version **uap10.0* for UAP specific libraries. For targeting specific versions of the UAP platform, you can specify the TPV in the folder name.

E.g, If you want to target RS1 version of the SDK, you can name the folder as the following

    \lib\uap10.0.10586.0\*
    \ref\uap10.0.10586.0\*

This nuget package is applicable to all projects who **TPV is either 10.0.10586.0** or **TPM>= 10.0.10586.0 && TPV<= 10.0.10586.0**. **ref** is given here for completeness and is only required if you have a reference assembly that is used to compile the app and there is a different implementation assembly in lib that is copied into the apps output.


If developers need to specify multiple versions of the assembly targeting specific versions of the SDK they can do by creating multiple libraries targeting specific versions of the OS. E.g,
    
    \lib\uap10.0.14393.0\*
    \lib\uap10.0.10586.0\*
    \ref\uap10.0.14393.0\*
    \ref\uap10.0.10586.0\*



 













