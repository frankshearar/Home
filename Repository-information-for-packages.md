## Issue
Link to the GitHub issue that tracks the work and discussion.

## Problem
Look at the issue description for the background and the problem details. In short, since mid 2016, the NuGet package manifest (.nuspec) has supported the <repository> XML element. Earlier this year - 2018, we documented the `repository` property. But there is no clarity on the purpose of this repository, how this should be lit up on the NuGet Gallery, VS client or the format in which this should specified in the nuspec.

## Who is the customer?
* All the NuGet package publishers who are using this field in a certain way or are looking for recommendations on how to use this functionality
* All package consumers who would like to know the source code repository to make certain decisions. For example, developers look at the repository to understand how active the project is or if the package is open source (so that it can be serviced by them if required), etc. 

## Evidence
* A total of 35,231 packages have been published with <repository> metadata.
* There are 6,539 distinct package IDs that use repository metadata.

Look at the **Usage on nuget.org** section of the [linked issue](https://github.com/NuGet/NuGetGallery/issues/4941) for more details.

## Solution

### Authoring 

### Display on nuget.org (short-term)

### Long term ideas on the display and usage 
