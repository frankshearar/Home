Status: **Reviewing**

## Issue
The work for this feature and the discussion around the spec is tracked here:

**2Factor Auth on NuGet Gallery [#3252](https://github.com/NuGet/NuGetGallery/issues/3252)**

## Problem
NuGet.org accounts are currently secured by a simple username/password combination or linked to a Microsoft Account that is similarly protected. We want to make it harder to compromise these accounts by two factor authentication.

## Who is the customer?
All NuGet package authors will be protected by a more enhanced layer of security for public NuGet.org packages

*Authors can still publish package using the existing API keys to NuGet.org. To create a new API key, they may require the advanced security layer via 2-FA, if enabled* 

## Key Scenarios
The key scenarios we want to enable are:
* **Deprecate NuGet.org password based accounts.** NuGet.org accounts do not support 2-FA that is critical for enhanced security for Microsoft ecosystems. At NuGet.org, we do not want to build additional 2-FA capability for NuGet.org password based accounts. Instead we would like to leverage existing Microsoft accounts and Azure Active Directory solutions to enable this security functionality. As part of the feature, transition to the new sign-in systems would be seamless. *NuGet.org already supports Microsoft account sign-ins for existing accounts*
* **Enable and encourage enhanced security of NuGet.org accounts using 2-FA**. We will not mandate 2-FA usage for all accounts. 
* **NuGet package authors belonging to Organizations with AAD can authenticate on NuGet.org via their AAD instance**. These AAD instances can have 2-FA enabled on them which NuGet.org will respect. *Eg. MSFT packages require mandatory sign-in through secured @microsoft.com accounts federated through Microsoft Organization on AAD similar to our admin accounts.*

* Gallery instances (other than NuGet.org) should be able to remain with basic auth using username/password sign-ins.

## Solution

### Personal NuGet.org accounts
We plan to deprecate NuGet.org accounts not linked to Microsoft Accounts and require authentication to NuGet.org accounts via Microsoft Accounts that are secured by 2-FA. We will also support AAD logins. 

_For AAD, the experience will be similar to Microsoft Accounts. Clicking on the "Sign in with Microsoft" will lead to a login screen that will redirect to an AAD login if the mail id entered is an AAD account. Nothing else changes._

### Organization Accounts

NuGet.org will introduce a light weight "Organization" concept as covered by spec: [Organizations on NuGet.org](https://github.com/NuGet/Home/wiki/Organizations-on-NuGet.org)

## Plan

The aim is to enable 2-FA in phases:

**Phase 1:**
* Enable Microsoft accounts and AAD logins as the default way to login or register on NuGet.org
* Recommend users to enable 2-FA for their accounts
* *NuGet.org password logins will exist but will not be promoted for sign-ins or new user registrations*

Default login:

![image](https://user-images.githubusercontent.com/14800916/30129446-c3a4bc6e-92fa-11e7-86f2-fb11cd87d9af.png)

Encourage linking from NuGet account settings:

![image](https://user-images.githubusercontent.com/14800916/30139690-4d15160a-9324-11e7-9e40-f8ab68a4b2c2.png)

Encourage 2-FA:

![image](https://user-images.githubusercontent.com/14800916/30129499-e646c7d0-92fa-11e7-9fb9-88dfe2e24432.png)


**Phase 2:**
* Deprecate NuGet.org password logins. Ask users to connect with Microsoft/AAD accounts
* Enable **Organizations** on NuGet.org. Spec: [Organizations on NuGet.org](TBD)

Deprecate NuGet.org password login with migration path to MSA/AAD:

![image](https://user-images.githubusercontent.com/14800916/30301386-505b12da-970f-11e7-812a-32cbdd2685f2.png)

**Phase 3:**
* Disable NuGet.org password sign-ins. Enforce all accounts to connect with Microsoft accounts or AAD.

NuGet.org password sign-in is disabled:

![image](https://user-images.githubusercontent.com/14800916/30339210-20a952f6-97a3-11e7-91ce-ffaae53ed0bc.png)

**Note**: Manage Organizations, Manage packages and Upload package are disabled

![image](https://user-images.githubusercontent.com/14800916/30339136-e8f0cbf0-97a2-11e7-9406-8ace75ce2188.png)