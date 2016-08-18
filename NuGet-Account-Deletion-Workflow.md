##  Problem
Today there is no self-service option for a user to delete his/her NuGet account. Even if we do a get a genuine request to delete an account, we don’t have a way to do it since that would result in the orphaned packages.

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

###Ghost Account (Deleted User)
1. The username for the ghost account will be "Deleted User"
2. The profile page for this user will not show any associated packages or related statistics.
3. Clicking on contact owner on a package that has been re-parented under the ghost account will take the user to the [Contact Us](https://www.nuget.org/policies/Contact) page

###Duplicate account username check
During new account creation, a check must be added to de-duplicate against deleted accounts as well. 

##Open Questions
1. What about the information contained in the nuspec inside the package?
2. What about the links on the orphaned package page - [Project Site]() and [License]()

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
3/ The co-owners of all packages get a notification stating that the account has been reactivated.




