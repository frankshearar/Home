**Status**: Review

## Issue
[Task for Specing](https://github.com/nuget/home/issues/5907) and [Task for execution](https://github.com/nuget/home/issues/5904)

## Problem
Currently there is no way for package authors to sign their packages. Signed packages help with authenticity and integrity of a package when it is being consumed by NuGet users. Further, in future, this command will also facilitate package sources to sign the package to prevent package tampering once it has been published.

## Who is the customer?
All NuGet package authors.

## Evidence
Part of the larger [package signing](https://github.com/NuGet/Home/wiki/Author-Package-Signing) effort.

##  Solution
We will add a first level command to NuGet.exe which will allow package authors and package sources to sign NuGet packages.

### Command Signature 
```
usage: NuGet sign [package] [options]

Signs a NuGet package.

argument:

package - Path to the package that needs to be signed.


options:

-OutputDirectory - Directory where the signed package should be saved. By default the original package is overwritten by the signed package.

-CertificatePath - Path to the certificate to be used while signing the certificate. 
The path can be a file path or a certificate path from the local store of the format cert:\certificate_context\certificate_store_name\certificate_thumb_print.

-CertificateSubjectName - String representing the subject name of the certificate used to search the default local certificate store for the certificate. 
The search is a case-insensitive string comparison using the supplied value, which will find all certificates with the subject name containing that string, regardless of other subject values.

-CertificateFingerprint - SHA-1 fingerprint of the certificate used to search the default local certificate store for the certificate.

-CertificatePassphrase - Password for the certificate, if needed. 
This option can be used to specify the password for the certificate.

-CryptographicServiceProvider - Name of the Cryptographic Service Provider which contains the Private Key Container.
This option, along with -KeyContainer, can be used to specify the private key if the certificate file does not contain one.

-KeyContainer - Name of the Key Container which has the Private Key.
This option, along with -CryptographicServiceProvider, can be used to specify the private key if the certificate file does not contain one.

-Timestamper - URL to an RFC-3161 compliant trusted time-stamping server.

-HashingAlgorithm - Hashing algorithm to be used while digesting the package files. Defaults to SHA512.

-RSASignaturePadding - RSA Padding scheme used to sign the package with an RSA certificate. Supported padding schemes are PKCS1-v1.5 and PSS.
This option can be used to specify the padding scheme if the certificate is not signed with either of the two supported schemes.

-Force - Switch to indicate if the current signature should be overwritten. By default the command will fail if the package already has a signature.

```

### Details about options
* The options `CertificatePath`, `CertificateSubjectName` and `CertificateFingerprint` are different options available to the user to specify a certificate.
* The `CertificatePath` option is used to uniquely identify a certificate. The parameter accepts a file path or a certificate path from the local store in the following format - `cert:\certificate_context\certificate_store_name\certificate_thumb_print`.
* `CertificateSubjectName` and `CertificateFingerprint` options can be used to search the local certificate store. While searching the local store, if more than one matching certificates are found then we should prompt the user with the options and ask them to provide a more specific filter. 
* In all the cases we should display the certificate being used.  
* Users will have 2 options for providing the private key to be used to sign the package. Primarily they could provide a certificate which contains a proivate key, in such case we will use that to sign the package. However, if the certificate does not contain a private key then the user can provide the `CryptographicServiceProvider` and `KeyContainer` values to be used to find the private key.  
* While providing `CryptographicServiceProvider` and `KeyContainer` values, the user must ensure that the resolved private key must match the certificate file passed. Else the sign command will fail.
* `Force` option can be used to specify if an existing signature should be overwritten. If this switch is not used then we should fail if there is an existing signature.

### Acceptable Certificate sources
The command will support for the following certificates sources - 
 1. Certificate file - Path to the certificate file on the local file system or a network share.
 2. Certificate store/keychain - A URI to the certificate in the local certificate store or keychain by using a URI format - `cert:\certificate_context\certificate_store_name\certificate_thumb_print`or `cert:\path_to_keychain\certificate_thumb_print`
 3. Hardware Security Module - Under Investigation.
 4. CNG/CSP - User can provide the Cryptographic Service Provider name and the key container name. [Sample Code](https://msdn.microsoft.com/en-us/library/system.security.cryptography.cspparameters(v=vs.110).aspx)

### Signing Atomicity
The sign command should be atomic in nature i.e. If the command fails then the original package should not be modified. Possible options for this - 
 1. Do not allow in-place signing and mandate the `-OutputDirectory` options. 
 2. Back up the original package and overwrite it back if signing fails midway. This can be costly for large packages.

### Validation on Sign
The sign command should perform the following validations before attempting to sign the package - 
 1. Validate that the package exists on disk and the process has Read/Write access to the package.
 2. If `-OutputDirectory` if passed, validate that the process has write access to the path.
 3. Validate that the user has supplied a valid certificate through all of the options.
 4. If the certificate is password protected, validate that the user supplied a password using the `-CertificatePassphrase` option.
 5. Validate that the resolved certificate is currently valid.
 7. Validate that the certificate contains a private key or the user has provided the `-CryptographicSignatureProvider` and `-KeyContainer` options.
 6. Validate that the user has passed a valid timestamper url using the `-Timestamper` option.
 8. Logical validation on all the supplied options.

### Corresponding commands

In future we would like to add support for the following platforms - 

* Dotnet CLI - `dotnet nuget sign <package_path> [Options]`

* MSBuild target - `msbuild /t:nugetsign <package_path> [Options]`

### Implementation Details
The sign command will do the following - 
 1. Check if the passed arguments are valid. If not then throw.
 2. Wrap them into a `SignArgs` object.
 3. Invoke ` var signResult = SignCommandRunner.Execute(SignArgs);`.
 4. If `signResult.Status == Failure`, then  show an error indicating the failure.

The SignCommandRunner will do the following - 
 1. Convert the package path into an in memory Package Object/stream.
 2. Convert the certificate into an in memory Certificate object.
 3. Pass the SignArgs to NuGet.Sign API.
 4. return result to the caller.

## Open Questions 

 **1.** Whats the best way to pass the certificate password to the sign command?  
    Currently the spec only accepts a commandline switch with clear text password. But we can also support having an 
    encrypted password stored in a nuget.config file.  

 **2.** What kind of validation will we do before signing?  
    We should spec out all the validations that will be done before signing.  

 **3.** Do we need to add retry mechanism for the timestamper service?  
  Number of retries to get a timestamp for the countersignature.  
  Delay (in seconds) between the retries to get a timestamp for the countersignature.

## Feedback
Please use the [tracking issue](https://github.com/NuGet/Home/issues/5907) to provide feedback or any questions that you might have. Thanks!