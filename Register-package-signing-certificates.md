Status: **Reviewing**

## Issue
The work for this feature and the discussion around the spec is tracked here:
**Register package signing certificates on NuGet.org [#5346](https://github.com/NuGet/NuGetGallery/issues/5346)**

Package signing master spec list can be found here: [[Package signing]] 

## Problem
As part of package signing effort, there needs to be a way for authors to submit signed packages to NuGet.org. NuGet.org should not accept any signed package to a given account unless the signature matches the one of the certificates (with public key) registered by the author to his/her account. 
Registering the certificate also provides additional layer of security to the packages submission process. With this feature, if a certificate is registered against an account, one would not only require an API key to push the package but also require the package to be signed using one of the registered certificates.

## Who is the customer?
NuGet package authors who would like to sign their packages with a CA signed certificate.

## Key Scenarios
The key scenarios we want to enable are:
* Ability to register one or more certificates.
* Ability to override the certificates to be used for a package that has more than one owners.

### Out of scope
* Current assumption is that only a CA signed certificate will be allowed to sign a NuGet package. The discussion around whether NuGet.org will allow self signed certificate is **out of scope** for this spec.

## Solution

### Register certificates

* If an author wishes to sign all his/her packages using a certificate, he/she would have to register the certificate on NuGet.org. 
* If no certificates are registered, the author can continue to upload unsigned packages. However, if they try to upload/push a signed package, the package won't be accepted unless the certificate used to sign the package is registered on NuGet.org against his/her account.
* If one or more certificates are registered against an author's account, all the new packages (new or updates) will have to be signed packages.
* Following details about the certificate or otherwise should be shown for each registered certificate:
  * Name
  * Thumbprint
  * Expiry date
  * Status - (Active, Expired, Revoked)
  * Issuer
  * Number of **author's** packages on NuGet.org that were signed with this certificate
  * A delete action button

Proposed screenshots (not the final ones): 
![image](https://user-images.githubusercontent.com/14800916/35362315-c8161ed6-0119-11e8-8772-ed367b483000.png)

![image](https://user-images.githubusercontent.com/14800916/35362338-dd2835f2-0119-11e8-9753-43a681a52c38.png)

![image](https://user-images.githubusercontent.com/14800916/35362394-06b2412e-011a-11e8-80f4-86e0ab5d4a00.png)

### Deleting/Removing registered certificates
* A registered certificate can be safely removed by clicking on the delete button if there were no packages uploaded to NuGet.org signed with that certificate. The row for the registered certificate will no longer be shown.
* If there were one or more packages pushed to NuGet.org, delete action will disable the row that shows the registered certificate but should not remove the row altogether.
![image](https://user-images.githubusercontent.com/14800916/35362656-0f7add10-011b-11e8-94cd-6ee0cc6a46d6.png)

## Certificate requirements for packages with more than one owner

If a package is owned by more than one owners, then the following situations arise:
1. One of the owners registers certificates while atleast one other owner did not register any certificate
2. All the owners registered certificates

### 1. Atleast one owner has registered certificates and atleast one owner doesn't

* **Default:** The package needs to be signed by any one certificate registered by an owner.
* This can be overridden to require **No** certificate by any of the owners i.e. an unsigned package can be submitted after this override:
   ![image](https://user-images.githubusercontent.com/14800916/35362976-a0697e8e-011c-11e8-9148-dc84d2109854.png)

### 2. All the owners have registered certificate(s)

* **Default:** The package needs to be signed by any one certificate registered by an owner.
* This can be overridden to require any specific owner's certificate. Submitting an unsigned package will not be possible in this case.
    ![image](https://user-images.githubusercontent.com/14800916/35362998-b8af5072-011c-11e8-8416-47200842b281.png)
