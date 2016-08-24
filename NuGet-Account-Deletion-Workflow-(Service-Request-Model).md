1. User clicks on "More Info" (Delete my Account)
![collapsed](https://github.com/NuGet/Home/blob/dev/resources/AccountDeletionWorkflow/Account%20Page%20-%20Delete%20account%20collapsed.png)
2. Dialog explaining the implications and a link to the documentation (for more details) is displayed
3. The list of child packages is displayed along with the language which conveys that by proceeding the user understands that he/she will be relinquishing ownership of these packages.
![Dialog explaining the implications and a link to the documentation](https://github.com/NuGet/Home/blob/dev/resources/AccountDeletionWorkflow/Account%20Page%20-%20Delete%20account%20expanded%20-%20request.png)
4. User fills out the form and clicks "Yes, delete this Account"
![request submitted](https://github.com/NuGet/Home/blob/dev/resources/AccountDeletionWorkflow/Account%20Page%20-%20Delete%20account%20expanded%20-%20request%20submitted.png?raw=true)
5. Admin team reviews the request
6. The account can be immediately deleted if one of the following conditions is met
 * No child packages exist
 * Child packages if any have co-owners (packages won't be orphaned when the account is deleted)
7. If the account has child packages at risk of being orphaned, the Admin team will work with the account owner to transfer the ownership of the package.
 * The admin team would have the SLA time-frame to affect the transfer of ownership
 * At the end of the SLA time-frame, if ownership has not be transferred, the packages must be re-parented under "Deleted Account"
8. After receiving the request to delete an account, the account must be deleted within SLA time-frame
9. The SLA time-frame is 60 days 


***

[[Account Deletion Workflow|NuGet-Account-Deletion-Workflow]]
 
