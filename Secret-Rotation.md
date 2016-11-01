#Background:#

`NuGet.org` requires a number of resources in order to function, for example `SQL servers` and `Azure storage accounts`. The `NuGet.org` backend uses `access keys`, `SQL logins`, and other forms of authentication to access these services.

These credentials are long and cryptic enough to be incapable of being brute-forced, but they are never changed. Consider the hypothetical situation in which an attacker gains access to one of our `SQL databases` through a secret our services use. The attacker keeps their access forever--the username and password they have acquired never becomes invalid, so they can continue to steal our data or silently hijack our service without countermeasure!

To make matters worse, consider that the longer a secret is valid, the greater the chance of it being compromised. By regenerating our secrets on a regular interval we can reduce the risk of someone gaining access to inner workings of our service.

#Requirements:#
We would like to create system in which the secrets that the NuGet.org backend uses change over time to reduce the likelihood of being compromised and the effect of a secret being compromised. Secrets must rotate on a regular interval without human intervention, and doing so must not require redeploying or restarting the services.

#Design:#
We will create a job that performs secret rotations on all of our services on a regular interval.

##Required Setup:##
We recently integrated `KeyVault` with our backend. In order to acquire its secrets, our services contact a `KeyVault` server with some credentials. This server then returns the secret it has stored. We can change the secrets that our services use by changing these values stored on the server. Each of our services must check `KeyVault` at a regular interval to refresh the secrets it is using.

To rotate our secrets, we must first have two secrets that are valid to access each `database`, `storage account`, or other resource. For example, we would have two logins for each `SQL account` our services use, and two access keys for each `storage account` our services use. With this setup, rotating the secrets consists of replacing the login or access key that is not in use and then updating `KeyVault` to store the new secret. As a result, the secret that was used the week before is still valid, so our services have plenty of time to refresh the secret they have rather than being denied access to our resources immediately. We currently only have a single valid account for most of our SQL accounts, so we must create new ones for the purpose of using them in our rotations.

Graphically, this can be shown as followed.

####Secret Rotation Process Figure####

![Secret Rotation Process Figure](https://github.com/NuGet/Home/blob/sb-secretrotationspecimages/resources/secretrotation/secret%20rotation%20diagram.png)

In this figure, if a secret has a line in a cell above a timestamp (T1, T2, etc…) then it is a valid credential for a resource at that time. At any time, two secrets are valid for the resource, and upon a rotation, an older credential is replaced with a newer one. If the line for a secret is highlighted, then it is the secret currently stored by `KeyVault`. The newest credential is always stored in `KeyVault`, because it will not be invalidated on the next rotation.

##Functionality of the Job:##

###Tags:###
The job will use `KeyVault`'s tagging system to determine what secrets to rotate and how to rotate them. Each `KeyVault` can have its own dictionary of tags, which are `KeyValuePair`s of `string`s. As such, we can use these tags to tell the job information. Additionally, this information will also be available to people on the team who are looking at the secrets and curious what they are for. The possible tags are detailed below.

*Job Tag Schema*

![Job Tag Schema](https://github.com/NuGet/Home/blob/sb-secretrotationspecimages/resources/secretrotation/tag%20schema.png)

The job will handle rotating two types of secrets--`storage access keys` and `SQL accounts`. There is a difference in the number of tags that are necessary for each rotation. The process for rotating each type of secret will be described below.

###Storage Account Access Keys:###
Rotating `storage accounts access keys` is relatively simple because `Azure` has most of the functionality built in. `Azure` storage accounts have two valid access keys at any time, and there is an easy API call to make to regenerate them.

There will be one secret in `KeyVault` for each storage account. This secret will store the primary access key, intended to be used by all our services at any moment. Upon rotation, the job will use an `Azure` AD app with a `ServicePrincipal`, authenticated with using a certificate, to make calls to the `Azure` API. The job will check the two access keys stored by `Azure` with a name specified by the tag `<StorageAccountName>`. It will regenerate the key that is *not* stored in `KeyVault` and then save the new access key in `KeyVault`.

The figure below goes through two rotations.

####Storage Account Access Key Rotation Figure####

![Storage Account Access Key Rotation First Iteration](https://github.com/NuGet/Home/blob/sb-secretrotationspecimages/resources/secretrotation/storage%20account%20rotation%201.png)

During the first rotation, we regenerate `AccessKey2` because it was not the primary key at the time. `AccessKey2`  becomes `AccessKey3`. We then store `AccessKey3` in `KeyVault` because it is new.

After second rotation:

![Storage Account Access Key Rotation Second Iteration](https://github.com/NuGet/Home/blob/sb-secretrotationspecimages/resources/secretrotation/storage%20account%20rotation%202.png)

During the first rotation, we regenerate `AccessKey1` because it was not the primary key at the time. `AccessKey1`  becomes `AccessKey4`. We then store `AccessKey4` in `KeyVault` because it is new.


###SQL Accounts:###
Rotating SQL accounts is much more difficult because it is not possible to have multiple logins for a single account. As a result, we must maintain two individual SQL accounts, both with their own login which consists of a unique username and password.

There will be four secrets in `KeyVault` for each SQL account. These include a secret for…
	• Primary Account Username
	• Primary Account Password
	• Secondary Account Username
	• Secondary Account Password

Each secret will be mapped to one of these by two tags, `<SqlType>` and `<SqlRank>`. `<SqlType>` describes whether or not the secret is a username or password, and `<SqlRank>` describes whether or not the secret is a primary or secondary account.

Each of these secrets will have a tag `<SqlPrimaryUserSecretName>`, which will store the name of the secret containing the primary account username. The job will group these secrets by this tag and verify that all of the necessary secrets have been found. The job will also verify that these secrets all share the same `<SqlServerUrl>` and `<SqlDatabaseName>`.

Once the job has gathered all of the secrets, it will log into the secondary account and change the password. It then stores this new login as the primary account in `KeyVault`, and stores the old primary as the new secondary account.

####SQL Account Rotation Figure####

![SQL Account Rotation First Iteration](https://github.com/NuGet/Home/blob/sb-secretrotationspecimages/resources/secretrotation/sql account rotation 1.png)

During the first rotation, we changed `Password2` because it was the password of the secondary account. `Password2` became `Password3`. We then stored this new login, `Username2`-`Password3` as the new primary new login, in the secrets `Prod_GalleryV2ReadOnly-Username` and `Prod_GalleryV2ReadOnly-Password`. We then stored the old primary login, `Username1`-`Password1` as the new secondary login, in the secrets `Prod_GalleryV2ReadOnly-UsernameSecondary` and `Prod_GalleryV2ReadOnly-PasswordSecondary`.

![SQL Account Rotation Second Iteration](https://github.com/NuGet/Home/blob/sb-secretrotationspecimages/resources/secretrotation/sql%20account%20rotation%202.png)

During the second rotation, we changed `Password1` because it was the password of the secondary account. `Password1` became `Password4`. We then stored this new login, `Username1`-`Password4` as the new primary new login, in the secrets `Prod_GalleryV2ReadOnly-Username` and `Prod_GalleryV2ReadOnly-Password`. We then stored the old primary login, `Username2`-`Password3` as the new secondary login, in the secrets `Prod_GalleryV2ReadOnly-UsernameSecondary` and `Prod_GalleryV2ReadOnly-PasswordSecondary`.

###Job Frequency:###
On prod, the job will run and rotate our secrets once a week. This will give our services plenty of time to refresh the secrets they are using while also not allowing credentials to exist on the server for too long. The job will run on int at the same interval to mimic the prod environment.

On dev, the job will run multiple times per day. Rotating secrets frequently on dev will test our code to guarantee that our backend does not break upon rotation.

The addition of secret rotation will create a restriction for deployments. A deployment must survive two runs of the secret rotation job before we know that it correctly handles rotating credentials. Before a deployment is promoted to prod, it must survive two secret rotations on either dev or int. If we put dev on a relatively fast cycle (12 hours or less), the amount of time required before we know a deployment handles secret rotation will be relatively low. We can also run functional tests on our dev environment after a rotation to verify everything is working correctly.

#Comments:#
- Needs to handle the same key being in multiple KeyVaults
- Needs to handle being run multiple times in the same cycle without rotating secrets multiple times.
- Should be able to recover from a bad state.
    - If the job fails?
    - Temporary secrets?