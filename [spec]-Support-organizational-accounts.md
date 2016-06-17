# Support organizational accounts

## Problem

The NuGet Gallery currently is scoped to having only one type of account: a user. Users can upload and manage their packages, as well as add/remove ownership for their packages.

Now what happens if an _organization_ tries to publish a package? Right now, organizations (or teams)like ASP.NET have created their own account that serves as a "organization" account. All packages they publish are owned by this account. If the organization wants to upload a new package, they have to either use the credentials for this account _or_ add a team member as a package owner and let that team member perform the upload.

There are several issues with this:

* A logistical issue: who is this shared account? Who owns it and who manages it?
* A practical issue: how are packages published? Does the organization need to share the credentials to be able to perform their work? Or does a team member always have to be added as a package owner in order to sucesfully publish the package?
* Ownership: who owns the package? NuGet.org users can't clearly distinguish this if a package is owned by both the organization as well as a team member. Is it trustworthy?
* Security: if the organization account is a regular account, it can also push packages. Is this intended? Or should it always be a team member pushing packages on behalf of the organization?

## Who is the customer?

The customer range is broad. A customer can be a large organization, like Microsoft, wanting to have their employees publish packages on behalf of an official organization account. It can be individual teams, like the ASP.NET team. It can be open source projects, where contributors can publish packages on behalf of the project organization.

## Solution

A solution to this issue would be to introduce the concept of an "organizational" account. Creating an organization would help centralize that organization's packages on NuGet.org. All packages would be owned by the organization, making it clear for any Nuget.org user who is the owner of the package.

An organizational account could be created by a team member. This team member can then add/remove other team members from the organization. In the Minimum Viable Product (MVP), all team members would be equal and can add/remove other team members from the organization.

The organization itself is a logical group and has no privileges on NuGet.org. This means it can never:

* Login to NuGet.org
* Push packages to NuGet.org
* Create or manage credentials such as API keys
* Modify package ownership on its packages

Once an organization is created, all of the organization's team members can perform package operations using their own account, on behalf of the organization.

This implies that the NuGet.org service should be able to distinguish if the team member is acting as the organization, or acting as the user itself. For example, a team member could push their own spare-time packages as well as organizational packages.

A simple solution to this would be to have the team member always upload *new* packages as themselves, and have them change ownership to the organization account via the NuGet.org UI. For existing packages, ownership is already determined and NuGet.org can easily publish new *versions* under the organization.

Perhaps a more user-friendly way of doing this would be to introduce 2 places where the team member should state its intent with regards to who is publishing the package:

* When uploading through the NuGet.org UI. During the package upload, the team member should be able to select whether the package will be published under their own name or the organization name.

* When uploading through the NuGet.org API, it should be possible to distinguish on whose behalf the user is publishing the package.

  * By setting the package source to include the organization name. E.g. `nuget.exe push <package> <api key> -Source https://www.nuget.org/api/v2/package?organization=<organization>`

  * By updating the NuGet client and adding a new switch that can be used during package publish, e.g. `-Organization <organization>`

  * *Note that if a package already exists and a new package is being uploaded by the team member, stating this intent would be optional, as the package ownership has already been established.*

## Beyond the Minimum Viable Product (MVP)

This section describes additional features to the organizational accounts.

* Access levels - An organization may want to have various access levels for its users. For example, one team member may manage organization members, while another may only push package updates.

* The NuGet.org codebase contains several interesting aspects like expiring API keys and external identity provider integration. Based on these, several other features can be added to augment the organizational account:

  * API key Policy enforcement - The organization may enforce its users to have API keys that have a shorter validity period (e.g. 1 day instead of the default 90 days).

  * Identity provider enforcement - The organization may enforce its team members to always having to use a specific identity provider, for example Azure Active Directory. Additionally, the organization could enforce group membership if the external identity provider supprts it. This would imply that the organization's team members have to be member of a sepcific security group before beign able to authenticate against NuGet.org.

## Discussion

There are many things that can be done with organizational accounts and ideally, there is a discussion on the above spec first. An issue has been created for this discussion: https://github.com/NuGet/NuGetGallery/issues/3084