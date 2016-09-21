# Multiple API Keys

## Issue

Feedback on the spec can be given on the following issue - https://github.com/NuGet/NuGetGallery/issues/3246

## Problem

Currently, NuGet.org users can only create a Single API key for all their packages. For large GitHub organizations, it is necessary that multiple API keys be created that be scoped to specific actions and packages to prevent a single leak from compromising all the packages. In addition, this enables us to hide the API keys after a one-time generation further reducing the risk and enabling users to create keys with specific privileges.

## Who is the customer?

Large GitHub organizations or users with multiple packages and contributors

## Evidence
* Security Push
* Feedback from customers during the Expiring API keys discussion

## Solution

The key user workflows we want to enable is the following

* Enable users to create multiple API keys with a name and expiration range similar to current API keys.
* Restrict privileges of API keys to one or more packages
* Restrict key privileges to specific NuGet.org actions like Push, Un-list and Update
* Notify Users on new key creation


