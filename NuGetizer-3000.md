## Overview

Creating NuGet packages is the main way to redistribute libraries for .NET and Xamarin applications. However, there is currently no integrated support in the IDEs or build tooling.

Creating packages generally involves setting up complex structures of build outputs, manually authoring a nuspec manifest, and invoking command-line tools. The experience is particularly bad for so-called “bait and switch” packages which provide a single reference assembly and multiple platform-specific implementations. Although some tools do exist that can help with this process, they do not provide a cohesive workflow and end-to-end story.

The goal of the NuGetizer 3000 project is to make it as easy as possible to create and publish a NuGet package, including bait-and-switch packages, and provide guidance to users along the way. The focus is on providing a good UX on top of established existing mechanisms.

The design is inspired by [NuProj](nuproj.net). It adds a new Package Project project type that builds NuGet packages, and is consistent with existing Visual Studio idioms such as project references and project options. A package project can reference several .NET library projects, and when built it creates a nupkg package that includes the output of those libraries in the correct subdirectories.

To simplify the common case where a NuGet is created from a single portable or platform-specific library, the ability to generate Nuget packages will be added to all library projects. However, multi-platform projects will be first class, and there will be several major features to specifically support creating bait-and-switch packages, including wizards to set up the structure using shared projects, and the ability to generate reference assemblies automatically.

The design also proposes several future improvements to further simplify NuGet authoring and consumption.

## Contents

* [Principles](https://github.com/NuGet/Home/wiki/NuGetizer-Principles)
* [Core Scenarios](https://github.com/NuGet/Home/wiki/NuGetizer-Core-Scenarios)
* [Core Features](https://github.com/NuGet/Home/wiki/NuGetizer-Core-Features)
* [Future Features](https://github.com/NuGet/Home/wiki/NuGetizer-Future-Features)