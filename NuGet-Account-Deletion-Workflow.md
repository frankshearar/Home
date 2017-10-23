##  Problem
Today there is no self-service option for a user to delete his/her NuGet account. Even if we do get a genuine request to delete an account, we don’t have a clean way to do it since that would result in orphaned packages.

**[Tracking Issue](https://github.com/NuGet/NuGetGallery/issues/3204)** - 
Please comment on the issue for any questions/suggestions/criticism/praise that you may have for this feature.

## Who is the customer?
Any person or entity which has registered and has a nuget.org account.

## Evidence
The volume of account deletion requests is showing an upward trend. We have received 5 requests in the past 60 days. Also, to comply with privacy guidelines we must allow the user to delete all personally identifiable information (PII) including their account.

## Solution

* [Phase 1 - Service Request Model](https://github.com/NuGet/Home/wiki/NuGet-Account-Deletion-Workflow-(Service-Request-Model))
* [Phase 2 - Self Service Model](https://github.com/NuGet/Home/wiki/NuGet-Account-Deletion-Workflow-(Self-Service-Model))

### Ghost Account (Deleted User)
1. The username for the ghost account will be "Deleted User"
2. Clicking on contact owner on a package that has been re-parented under the ghost account will take the user to the [Contact Us](https://www.nuget.org/policies/Contact) page

### Contact Us Page
We will update the contact us page with details around Deleted user contact. i.e. the NuGet team will only respond to copyright license issues and we do not support individual packages.

### Deleted account history and Duplicate account username check **[checking with Legal on this]**
1. Usernames of all deleted account must be stored. They must be stored in a way that does not allow us to retrieve them directly (similar to passwords) but does allow us to block someone from creating new accounts with the same username.
2. During new account creation, a check must be added to de-duplicate against usernames of deleted accounts. 

### SLA
As a matter of policy, we will commit to a 60 day time-frame to complete the operation i.e. brokering transfer of ownership of packages at risk of being orphaned, removing association from packages, re-parenting the packages, and deleting the account. Our privacy policy will be updated to stipulate the same.

## Open Questions
1. What about the information contained in the nuspec inside the package?
  * Legal - From the legal side, since we already say that any personal information that you put into the package will be public information, I do not think we have to change the privacy statement in that aspect 

## Feedback
**[Tracking Issue](https://github.com/NuGet/NuGetGallery/issues/3204)** - 
Please comment on the issue for any questions/suggestions/criticism/praise that you may have for this feature.




