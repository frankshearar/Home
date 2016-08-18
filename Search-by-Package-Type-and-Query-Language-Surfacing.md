## Revisions

- **2016-08-16** - Initial spec.

# Search by Package Type and Query Language Surfacing

## Problem
With the introduction of [Package Types](https://github.com/NuGet/Home/wiki/Package-Type) to NuGet Packages, authors are now able to specify if a package should be installed under the `"tools"` or `"dependencies"` node in a project.json. With this new classification, comes the necessity to be able to limit search results to only packages that are `"tools"` or `"dependencies"`. Currently, the NuGet Search API provides certain search operators to returns results that match certain IDs, Authors, etc. This spec details the addition of a new operator for Package Type, as well as a feature to surface the query language for the NuGet Search API. 

## Who is the customer?
Many NuGet Package consumers utilize the search function of the website and client (both use the same service) to look for packages to use. This features is aimed at any NuGet consumer that wants to find specific package types for their development.

Additionally, the addition of features to surface the query language for the NuGet Search API will be for any consumer who wants to have more powerful search.

## Evidence
Through our customer development interviews, we found that many NuGet customers wanted more a more powerful search to be able to find the packages they needed for their development. Of the 25 NuGet customers we talked to, 13 out of 15 (86.7%) applicable consumers expressed a want for searching by publisher or author. This functionality is already in place with the Search API, customers are just not aware of it. Users want to have more specific searches with filtering by author, owner, and other factors, we need to empower them to do so. 


## Solution
### Project Type Search Tag
The enable searching by Package Type, users should be able to type `PackageType:[TYPE]` into the search bar. The `PackageType` tag will be case-sensitive.

A user can specify anything to be a Package Type, though only two are officially supported (`"DotnetCliTool"` and `"Dependency"`). As such, this search tag sill support the searching of any string for Project Type and return the applicable results. If a package does not have a Package Type specified in in the .nuspec, the package is automatically assumed to be a `"Dependency"` type. 

Searching with a type will return all packages of that specified type. The `[TYPE]` string is not case-sensitive.

