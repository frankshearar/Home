##  Problem
Today there is no self-service option for a user to delete his/her NuGet account. Even if we do get a genuine request to delete an account, we don’t have a clean way to do it since that would result in orphaned packages.

**[Tracking Issue](https://github.com/NuGet/NuGetGallery/issues/3204)** - 
Please comment on the issue for any questions/suggestions/criticism/praise that you may have for this feature.

## Who is the customer?
Any person or entity which has registered and has a nuget.org account.

## Evidence
The volume of account deletion requests is showing an upward trend. We have received 5 requests in the past 60 days. Also, to comply with privacy guidelines we must allow the user to delete all personally identifiable information (PII) including their account.

##Solution
Add a "Delete Account" section under  https://www.nuget.org/account that triggers the following workflow:

1. Clicking on more info expands the Delete account section (similar to profile picture, by clicking 'more info')
2. Clearly states the consequences of proceeding with this action (along with a link to a new doc with detailed information)
3. User has to type the phrase "delete my account" in the box provided for it
4. User has to type the password
5. Click on "Delete my account" button

If the phrase and password check pass:

1. The account is deleted immediately - the user is logged out and will no longer be able to login with the same credentials
2. https://www.nuget.org/profiles/username redirects to the ghost account profile
3. the username is stored in a non-retrievable form (this is to ensure compliance with privacy guidelines)
4. The account will be removed as the owner from all associated packages
5. All orphaned packages will be re-parented under a ghost account and the author field will be overwritten with "Deleted User"
6. The co-owners get an email stating that an owner account has been deleted and hence removed as one of the owners of the package.

![collapsed](https://github.com/NuGet/Home/blob/dev/resources/AccountDeletionWorkflow/Account%20Page%20-%20Delete%20account%20collapsed.png)

![expanded](https://github.com/NuGet/Home/blob/dev/resources/AccountDeletionWorkflow/Account%20Page%20-%20Delete%20account%20expanded.png)

###Ghost Account (Deleted User)
1. The username for the ghost account will be "Deleted User"
2. The profile page for this user will not show any associated packages or related statistics.
3. Clicking on contact owner on a package that has been re-parented under the ghost account will take the user to the [Contact Us](https://www.nuget.org/policies/Contact) page

###Contact Us Page
We will update the contact us page with details around Deleted user contact. i.e. the NuGet team will only respond to copyright license issues and we do not support individual packages.

###Deleted account history and Duplicate account username check
1. Usernames of all deleted account must be stored. They must be stored in a way that does not allow us to retrieve them directly (similar to passwords) but does allow us to block someone from creating new accounts with the same username.
2. During new account creation, a check must be added to de-duplicate against usernames of deleted accounts. 

###SLA
Ideally, the account should be deleted immediately. However, to provide room for error, as a matter of policy, we will commit to a 5 business day time-frame to complete the operation i.e. deleting the account, removing association from packages and re-parenting the packages. Our privacy policy will be updated to stipulate the same.

##Open Questions
1. What about the information contained in the nuspec inside the package?
  * Legal - From the legal side, since we already say that any personal information that you put into the package will be public information, I do not think we have to change the privacy statement in that aspect 
2. What about the links on the orphaned package page - [Project Site]() and [License]()?

##Feedback
**[Tracking Issue](https://github.com/NuGet/NuGetGallery/issues/3204)** - 
Please comment on the issue for any questions/suggestions/criticism/praise that you may have for this feature.

##Solution - Cadillac version
Below is the advanced workflow with the cooling period safeguard that gives the user 'x' number of days to change his/her mind. Based on feedback, we can consider investing in implementing this.

###Delete
1. When clicked, if the account has associated packages
  * If the account being deleted is the only owner, provide information about adding co-owners. The user can choose to not add a co-owner and we should provide information that in this case, the package will be re-parented under a <deleted account>
  * If the associated package has additional authors, the account being deleted is simply removed from the list of owners for that package. The co-owners get a notification that an account has been marked for deletion.
2. Next, ask for a confirmation with a clear message that the user understands the implication (see Legal section below) and that he/she has 30 days to change their mind.
3. When the user confirms, mark the account for deletion and display the date when the deletion would happen.
4. When marked for deletion, we freeze all activity which means the user can’t change any account settings or push packages.
5. The account page is greyed out with the message that the account is marked for deletion along with the date on which that will happen and a button to “Reactivate account”.

### Re-activate

1. If the user clicks “Reactivate account”, ask confirmation and display a message with the implications.
2. If the user confirms, all functionality is restored.
3. The co-owners of all packages get a notification stating that the account has been reactivated.




