##  Problem
Today there is no self-service option for a user to delete his/her NuGet account. Even if we do get a genuine request to delete an account, we don’t have a clean way to do it since that would result in orphaned packages.

**[Tracking Issue](https://github.com/NuGet/NuGetGallery/issues/3204)** - 
Please comment on the issue for any questions/suggestions/criticism/praise that you may have for this feature.

## Who is the customer?
Any person or entity which has registered and has a nuget.org account.

## Evidence
The volume of account deletion requests is showing an upward trend. We have received 5 requests in the past 60 days. Also, to comply with privacy guidelines we must allow the user to delete all personally identifiable information (PII) including their account.

##Solution

* [Phase 1 - Service Request Model](/NuGet-Account-Deletion-Workflow-(Service-Request-Model))
* [Phase 2 - Self Service Model](https://github.com/NuGet/Home/wiki/NuGet-Account-Deletion-Workflow-(Admin-View))




