Status: **Reviewed**

## Issue
The work for this feature and the discussion around the spec is tracked here:

**Organizations on NuGet.org [#4627](https://github.com/NuGet/NuGetGallery/issues/4627)**

## Problem
NuGet.org today does not support a concept of organizations - teams or group of people owning and managing packages. Users workaround this in one of the following ways:
* Create a Distribution List alias (with email-address) with the required people on it. And use this email address for creating an account on NuGet.org
* Have multiple authors co-own packages. This becomes un-manageable as the number of owners increase.

With the introduction of [2-FA feature](https://github.com/NuGet/NuGetGallery/issues/3252), NuGet.org needs a better way to manage a organizations for team/group owned packages without compromising the enhanced security. For example, as discussed above, there are many Distribution List (DL) aliases used as individual accounts on NuGet.org to manage team packages. Since the email id associated with this account is of a DL, it cannot have a way to sign in using MSA/AAD or a way to authenticate using 2-FA.

Organizations can also serve as a central way to manage various policies on packages. *Eg. Enforce 2-FA accounts for all members or package co-owners or enforce signed packages for the organization owned packages.*

## Who is the customer?
Enterprise NuGet package authors who want to manage packages as a team and not just using an individual accounts. Eg. <TBD - ASPNET> 

## Key Scenarios
The key scenarios we want to enable are:
* Enable authors to create new organizations
* Enable authors to manage the organizations and its members
* Enable authors to easily manage packages for an organization
* Enable policies on organization and its packages
* Enable authors to transform their existing accounts to Organizations

## Solution
The idea is to have a **light-weight Organization** concept on NuGet.org. Organization names would be unique and non-conflicting with owner names. Details of the proposed solution is discussed below:

### Enable authors to create new organizations 

A new organization can be created from "Manage Organizations" option:
![image](https://user-images.githubusercontent.com/14800916/30187514-cd09f8ca-93de-11e7-88c4-8e3a54630d21.png)

It would take the following properties:
* **Name**: This is similar to owner usernname and needs to be unique across NuGet.org.
*Once an organization is created, the name cannot be changed.*
* **Email**: Email address used for communication for packages' support - 'Contact owners'. It will also be used for the organization icon/image using gravatar.com.
* **Logo**: Similar to owner profile pic. This will be a gravatar associated with the org email id.

![image](https://user-images.githubusercontent.com/14800916/30303819-1e340d6a-971f-11e7-80bd-8fa7928c10f0.png)

### Enable authors to manage organizations and its members

* Organizations will have two types (roles) of memberships:
   * **Admin**: As the name states, this role implies all privileges. An admin can
      * Upload new package
      * Edit, delete or update existing packages. 
      * Add/remove other members (including other Admin members).
      * Accept package co-ownership requests 
   * **Collaborator**: A collaborator will have limited permissions. A collaborator can 
      * Edit, delete (unlist) or update packages 
      * **Cannot** perform rest of the operations that Admins can.

* A member can leave an organization only if there are other members (or Admins) in the organization.
* A member can delete an organization only if the organization does not own any package(s) and if the member is the only member of the organization. 
  * The namespace for the deleted organization is released and can be taken up as an account or a new organization for others. 

### Enable authors to easily manage packages for an organization

#### Managing packages from "Manage packages" page

There would be a filter for organizations to be able to filter the packages belonging to the individual owner or an organization the owner is a member of. By default "All packages" will be shown.


![image](https://user-images.githubusercontent.com/14800916/30302514-64f7c858-9716-11e7-990b-28d8850fbb71.png)


#### Direct upload of packages

* The owner property will be the top most property shown to verify. This can be changed before publishing. The author/owner will be the default owner of new packages that can be changed using a drop-down. For existing packages, this field will be non-editable.
![image](https://user-images.githubusercontent.com/14800916/30301544-32c69e14-9710-11e7-9f22-b58e99e6d4d4.png)

#### Publishing packages using CLI (push)

* If an existing package is being pushed, there is no change here.
* If a new package is being pushed, the API key will decide whether the package being pushed is meant for an organization and if so, which one.

**Note**:
* All API keys are owned by an author. There is no separate API key management for an organization.
* API keys can be scoped to organizations. At a time, an API key can be meant for either the author's (owner's) packages or **one of the organizations**' packages. i.e. even if the author/owner specifies "*" in the glob pattern, it means "all packages" but scoped to the entity (individual or organization) as specified in the entity drop-down.

#### What happens to author's existing API keys?
* All the existing API keys are scoped for his/her account packages.
  * Any new package upload with existing API keys will be uploaded to his own account and not any of the organizations he/she creates
  * To upload new packages against an organization, the author will have to create new key scoped to an organization.

#### What happens when an author is leaves an organization (on NuGet.org)?
* All the API keys scoped to the organization will cease to work.
* The author cannot upload a package against the organization he just left or was removed from.

Organization scope while creating an API key:

![image](https://user-images.githubusercontent.com/14800916/30302450-efba9f98-9715-11e7-9dc5-0b11bb05fccd.png)


### Enable policies on organization and its packages

With organizations, it should be easy to implement various policies on packages and its owners. For example, the following policies could be implemented on packages owned by an organizations:
* Enable enhanced security via 2-FA for all members of the organization
* Enable enhanced security via 2-FA for all organization package owners. This means if a non-member (of the org) co-owns a package, then this policy would apply to that owner as well.
* Enable all organization owned packages to be signed

Policies on the organization page:

![image](https://user-images.githubusercontent.com/14800916/30302173-2297da86-9714-11e7-9160-0e6587ec67a8.png)

Policies applied to an author will be visible on the account page with the reason.

![image](https://user-images.githubusercontent.com/14800916/30302423-c2fc0b86-9715-11e7-8ae7-065879c751bc.png)

### Enable authors to transform their existing accounts to Organizations

There should be a way to transform existing owner accounts to organizations. This would be helpful for existing accounts that are today proxies for an organization, team or a group of people working together.

Possible entrypoint for this option:
![image](https://user-images.githubusercontent.com/14800916/31681107-209f85fc-b32b-11e7-9a49-220944af377a.png)
