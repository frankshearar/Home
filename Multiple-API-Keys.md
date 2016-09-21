# Multiple API Keys

## Problem

Currently, NuGet.org users can only create a Single API key for all their packages. For large GitHub organizations, it is necessary that multiple API keys be created that be scoped to specific actions and packages to prevent a single leak from compromising all the packages. In addition, this enables us to hide the API keys after a one-time generation further reducing the risk and enabling users to create keys with specific privileges.

## Who is the customer?

Large GitHub organizations or users with multiple packages and contributors

## Evidence
_What is the evidence that behaves us to act?_
_Evidence can be impassioned tweets or mails, mile long conversations in issues, rants on blogs, sweet sweet data from telemetry!!! If you can show pain, you can rally the troops._

## Solution
_Detailed explanation of the solution. The more pictures/code snippets based on the feature the merrier. Pictures keep folks awake when reading specs._