## Issue
[#7594](https://github.com/NuGet/NuGetGallery/issues/7594) - Surface PackageType query parameter in the NuGet.org search API

## Problem
When searching for packages, it is not possible to limit results to a specific PackageType.

## Who is the customer?
dotnet.exe - the ability to search for dotnet CLI tool only packages on NuGet.org

## Evidence
Today, the state-of-the-are is this manually maintained list - https://github.com/natemcmaster/dotnet-tools

## Solution
* Addendum to NuGetSearch Protocol - PackageType
* This is an extension to the protocol defined at https://docs.microsoft.com/en-us/nuget/api/search-query-service-resource
* Current usage of package types:
		PackageTypeName	Count
		(empty)	1855381
		DotnetTool,	6590
		Template,	4046
		DotnetCliTool,	1209
		MSBuildSdk,	758
		Dependency,	526
		AzureSiteExtension,	463
		DotnetPlatform,	213
		bugshooting.plugin.v3.output,	23
		TestPackageType,	7
		Passby.Microservice.Template,	6
		MicroServiceTemplate,	6
		WindowsAdminCenterExtension,	3
		aspnetcoreapi,	2
		Template,Template,	2
		Hello,Hello2,Hello3,	1
		BoostedLib,	1
		Native,	1
		CodingTool,	1
		DotnetCliTool,Dependency,	1
		Data,	1
		

## Non-goal:
* We will NOT support queries by packageTypeVersion. This includes returning it in the search response as of now.
* PackageType will not be forwarded to the registration. There are currently no scenarios that require this.

1. Request Protocol Updates:
	GET {@id}?q={QUERY}&skip={SKIP}&take={TAKE}&prerelease={PRERELEASE}&semVerLevel={SEMVERLEVEL}&packageType={PACKAGETYPE}

	"packagetype" field MAY be provided to further filter results.

	If "packagetype" is provided, {PACKAGETYPE} MAY BE provided and SHOULD BE any string.
		If {PACKAGETYPE} is not a valid package type as per https://github.com/NuGet/Home/wiki/Package-Type-%5BPacking%5D, an emtpy result SHOULD BE returned.
		If {PACKAGETYPE} is empty, no filter will be applied.
			Passing no value to the packageType parameter will behave as if the parameter was not passed.
	
	Response will contain packages for which the LATEST VERSION has at least one packageType that matches {PACKAGETYPE}.
		LATEST VERSION is the latest version as defined by the criteria given in the query. Dependent on values of:
			prerelease={PRERELEASE}
			semVerLevel={SEMVERLEVEL}

2. Search Response Updates
	2.1 Response Shape update:
	{
		"totalHits": int,
		"data": [
			"id": string,
			... (further fields omitted for brevity. See https://docs.microsoft.com/en-us/nuget/api/search-query-service-resource for complete list)
			"verified": boolean,
			"packageTypes": [
				{
					"name": string
				}
			]
		]
	}
	
	2.2 Description
	"packageTypes" MUST ALWAYS be present.
	"packageTypes" MUST HAVE a length of at least 1 (one)
	Name MUST BE a valid package type as specified by https://github.com/NuGet/Home/wiki/Package-Type-%5BPacking%5D

3. Catalog Protocol Updates
	3.1 Catalog leaf Shape updates:
	Package details catalog items will be the only update
	{
		"authors": string,
		... (further fields omitted for brevity. See https://docs.microsoft.com/en-us/nuget/api/catalog-resource for complete list)
		"verbatimVersion": string,
		"packageTypes: [
			{
				"name": string,
				"version": string
			}
		]
	}

	3.2 Description
	"packagetypes" MUST NOT be present if nuspec does not specify any (this includes implied dependency)
	If "packagetypes" is present, it MUST BE of length AT LEAST 1 (one)
	Name MUST BE a valid package type as specified by https://github.com/NuGet/Home/wiki/Package-Type-%5BPacking%5D
	Version MAY BE present IF and ONLY IF it was specified in the corresponding package nuspec
	
	Currently (2019.12.13), packageTypes node in catalog will either be not present (no packageTypes node in nuspec), or an empty string (packageTypes node present in nuspec).
