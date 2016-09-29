## Issue
Spec for the work is available here: - https://github.com/NuGet/Home/issues/3561

## Problem
Currently there are multiple ways (packages.config, project.json and PackageReference in csproj) of managing dependencies and we want to standardize on one global way of managing dependencies. Its hard on customers to read and understand about multiple ways of managing dependencies. 

## Who is the customer?
All .NET Customers who want to move towards managing dependencies via the model built for project.json.

## Solution
_Detailed explanation of the solution. The more pictures/code snippets based on the feature the merrier. Pictures keep folks awake when reading specs._