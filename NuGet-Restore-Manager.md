# NuGet Restore Manager

- [ ] How to bootstrap N.R.M. on project open or new?
	- Mef component
	- Second packagedef in vsix
- [ ] How to get info from VS at bootstrap time? 
	- Packages.config - project directories, packages directory
		- NuGet.Config discovery is expensive
	- UWP Project.json - dg info (needs to keep updated)
- [ ] *any* changes watcher
	- Project.json
	- Settings changes
	- Dependency changes / dg info for uwp (check if we need this as separate update)
	- Lock file presence
- [ ] CPS "Restore Nominator" Capability - csproj integration, capabilities?
	- When nominating CPS supplies project dir, intermediate dir, restore output type (uap, netcore), dg graph
- [ ] How to block the build until we have full info from VS?
	- Virtual Project?


## Problem
_What is the problem(s) we are trying to solve? Why is it a problem. What horrible workarounds are we subjecting our users too._

## Who is the customer?
_Who is the customer that is running into the problem. Which customers would dance for joy and donate to save the space unicorns foundation on getting this feature. Customers here could be individuals, nuget customer segments (package authors, consumers), enterprises, partners within Microsoft, external partners etc..._

## Evidence
_What is the evidence that behaves us to act?_
_Evidence can be impassioned tweets or mails, mile long conversations in issues, rants on blogs, sweet sweet data from telemetry!!! If you can show pain, you can rally the troops._

## Solution
_Detailed explanation of the solution. The more pictures/code snippets based on the feature the merrier. Pictures keep folks awake when reading specs._