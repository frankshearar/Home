Status: **Incubation**

## Issue
Issue for spec - [NuGet/Home#6419](https://github.com/NuGet/Home/issues/6419)  
Parent spec - [Repository-Signatures](https://github.com/NuGet/Home/wiki/Repository-Signatures)

## Problem
Once we enable package signing, we need to enable consumers to be able to trust a package repository. Further, that trust information needs to be stored into the users machine.

## Who is the customer?
All NuGet package consumers.

## Key scenarios
* Enable package consumers to store repository trust information

## Solution
* Update the schema for nuget.config file to be able to store repository trust information.
* Define a gesture for users to be able to trust a package repository.

This spec tackles the first half of the solution i.e. Update the schema for nuget.config file to be able to store repository trust information.







