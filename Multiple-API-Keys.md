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

The key scenarios we want to enable is the following

* Enable users to create multiple API keys with a name and expiration range similar to current API keys.
* Restrict privileges of API keys to one or more packages
* Restrict key privileges to specific NuGet.org actions like Push, Un-list and Update
* Notify Users on new key creation
* Maintain Legacy API keys, only new API keys have support for privileges, expiration and scope.
* Manage API key - Ability to delete any API key including Legacy

### User Workflow

* User Logs into NuGet.org
* Goes to Account Details
* Clicks on Show details in the API Keys section
* Navigates to Generate API Key
* Adds a Name for the the Key
* Selects the expiration interval (by default set to 365)
* From the list of packages (all packages owned by the user) selects the packages that the key will apply to (by default none is selected)
* From the list of following scopes Push, Unlist, Update selects privileges for the key (by default none will be selected)
* Clicks on Generate API key
* Notification is sent to registered email Id about the key create with name, packages and scope information.\
* Copies the API key and saves it in NuGet.config or uses it for a one-time operation

### Screens

**Create API Key - Expiration,Name, Packages and Scopes** 

In the interest of space, I have omitted the examples section in the below screen. We should not remove this from the current UI

![](https://github.com/NuGet/Home/blob/dev/resources/MultipleAPIKeys/InitialMultipleAPIKeys.png)

**Generate API Key**


**Manage API Keys** 



