Status: **Incubation**
2FA = two-factor authentication also referred to as two-step verification.

## Issue
The work for this feature and the discussion around the spec is tracked here:
**Organization policies and 2FA settings [#5599](https://github.com/NuGet/NuGetGallery/issues/3252)**

## Who is the customer?
* All NuGet package authors who want to enable an additional layer of security for their accounts
* All NuGet package authors who be protected by a more enhanced layer of security for public NuGet.org packages
* All NuGet package authors who wish to publish signed packages.
* All NuGet.org Organization admins who want specific settings/enforcement on members and packages.

## Key Scenarios
Here are the 2FA related requirements:
As Noel a NuGet.org user,
* I should be able to enable 2FA to sign in to my account for enhanced security.
* I am required to use 2FA to sign in to my account if I want to manage certificates.
* I should be able to enable 2FA sign-in for all users who wish to manage packages for an Organization I administer. This includes:
* * Manage certificates for Organization – Add/Remove/Override
* * Manage API keys scoped to Organization
* * Upload/update organization packages

Additional Organization policies:
As Noel a NuGet.org user, who is an admin of an Organization on NuGet.org,
* I should be able to enforce membership to only my company’s employees - AAD based accounts belonging to the same tenant
* I should be able to require specific metadata for packages uploaded/updated for my Organization (on NuGet.org)

## Solution

### Enable 2FA setting for individual accounts
### Enable 2FA policy setting for Organization's members
### Other Organization policies (P2)
### 2FA requirement for registering certificates
