Status: **Incubation**

## Issue
The work for this feature and the discussion around the spec is tracked here - **2Factor Auth on NuGet Gallery [#3252](https://github.com/NuGet/NuGetGallery/issues/3252)**

## Problem
NuGet.org accounts are currently secured by a simple username/password combo or linked to a Microsoft Account that is similarly protected. We want to make it harder to compromise these accounts by requiring mandatory two factor authentication on accounts linked to NuGet.org

## Who is the customer?
All NuGet package authors and the community who will be protected by a more enhanced layer of security for public NuGet.org packages

## Key Scenarios
The key scenarios we want to enable are given below:

* NuGet.org only supports sign-in through Microsoft Accounts (or possibly GitHub) secured that are secured via 2-FA.
* NuGet package authors belonging to Organizations with AAD can authenticate on NuGet.org via their AAD instance. These AAD instances need to have 2 FA enabled on them. Eg. MSFT packages require mandatory sign-in through secured @microsoft.com accounts federated through Microsoft Organization on AAD similar to our admin accounts.

**[Open]** How does authentication work for existing team alias logins? How do we migrate such accounts?

## Solution
More info coming soon.

### Personal NuGet.org accounts

Current thinking is that we will deprecate NuGet.org accounts not linked to Microsoft Accounts and require authentication to NuGet.org accounts via Microsoft Accounts that are secured by 2 FA. We will prevent Microsoft Accounts being linked that are not secured by 2 FA.

In addition to this, we will consider adding more authentication providers like GitHub and Twitter in the future that are constrained by the principle outlined above.

### Organization Accounts

Users can choose to authenticate to NuGet.org using their work accounts provided these are federated to Microsoft AAD secured by 2 FA. 



