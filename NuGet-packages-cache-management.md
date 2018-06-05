## Issue
A better cache expiration policy [#4980](https://github.com/nuget/home/issues/4980)

## Problem
The user local NuGet cache has a tendency to grow unconditionally with limited option to clean it up. The only option available today is to nuke the full cache with the [`nuget locals`](https://docs.microsoft.com/en-us/nuget/tools/cli-ref-locals) command.

## Who is the customer?

## Evidence

## Solution
