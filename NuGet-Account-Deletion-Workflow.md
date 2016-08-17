##  Problem
Today there is no self-service option for a user to delete his/her NuGet account. Even if we do a get a genuine request to delete an account, we don’t have a way to do it since that would result in the orphaned packages.

## Who is the customer?
Any person or entity which has registered and has a nuget.org account.

## Evidence
The volume of account deletion requests is showing an upward trend. We have received 5 requests in the past 30 days. Also, to adhere to privacy laws we must allow the user to delete all personally identifiable information (PII) including their account.

##Solution
###Delete
Add a "Delete Account" button under  https://www.nuget.org/account that triggers the following workflow:

1. When clicked, if the account has associated packages
  a. If the account being deleted is the only owner, provide information about adding co-owners. The user can choose to not add a co-owner and we should provide information that in this case, the package will be re-parented under a <deleted account>
 b. If the associated package has additional authors, the account being deleted is simply removed from the list of owners for that package. The co-owners get a notification that an account has been marked for deletion.
2. Next, ask for a confirmation with a clear message that the user understands the implication (see Legal section below) and that he/she has 30 days to change their mind.
3. When the user confirms, mark the account for deletion and display the date when the deletion would happen.
4. When marked for deletion, we freeze all activity which means the user can’t change any account settings or push packages.
5. The account page is greyed out with the message that the account is marked for deletion along with the date on which that will happen and a button to “Reactivate account”.

### Re-activate

1. If the user clicks “Reactivate account”, ask confirmation and display a message with the implications.
2. If the user confirms, all functionality is restored.
3/ The co-owners of all packages get a notification stating that the account has been reactivated.

###Ghost Account (Delete User)
design of the ghost account
name of the ghost user
associated email address

###Duplicate account username check
We must preserve uniqueness of the username name across all accounts that were ever created on nuget.org including accounts that have been deleted. For example, if I create an account with the username "karann-msft" and delete it, no one should ever be able to create an account with the same username.




