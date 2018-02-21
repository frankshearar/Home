Status: **Implemented**

## Issue
The work for this feature and the discussion around the spec is tracked here - **Enable NuGet Account Deletion workflow [#3204](https://github.com/NuGet/NuGetGallery/issues/3204)**


##  Problem
Today there is no option for a user to delete their NuGet account. If we get a valid request to delete an account, we don’t have a workflow to do it.

## Who is the customer?
Any person or entity which has registered and has a NuGet.org account.

## Solution

### User workflow
1. Go to https://www.nuget.org/account/delete (URL subject to change). You are prompted to login, even if you already are.

2. Form to submit the request

![](https://github.com/NuGet/Home/blob/dev/resources/AccountDeletionWorkflow/v2/Account_delete_01.png)

3. Click submit and confirm

![](https://github.com/NuGet/Home/blob/dev/resources/AccountDeletionWorkflow/v2/Account_delete_02.png)

4. Request submitted

![](https://github.com/NuGet/Home/blob/dev/resources/AccountDeletionWorkflow/v2/Account_delete_03.png)

* This page will be displayed if you go to delete URL after the request has been submitted.

5. You receive an email notification stating that we have received a request to delete your account.

6. Account is deleted and the account page leads to a 404. All orphaned packages are unlisted.

* The account can be deleted immediately if no child packages exist, or all child packages have co-owners (packages won't be orphaned when the account is deleted). Else the support team will reach out to the current owner for a recommendation for an alternate owner. 

### Packages with no owners

1. The package is unlisted and NuGet.org displays a warning under owners.

![](https://github.com/NuGet/Home/blob/dev/resources/AccountDeletionWorkflow/v2/Account_delete_05.png)

2. Contact Owners Page

![](https://github.com/NuGet/Home/blob/dev/resources/AccountDeletionWorkflow/v2/Account_delete_06.png)

### Duplicate account username check
1. Username of a deleted account will be reserved and new accounts with the same username cannot be created. Attempting to create a new account with the same username will display the same message we show today if there is an existing account with that username.
2. When a deleted user tries to login will display the same message we show today for incorrect username and/or password.



