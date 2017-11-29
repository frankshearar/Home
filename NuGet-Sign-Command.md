**Status: Reviewed** 

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
usage: NuGet sign <package_path> -Timestamper <timestamp_server_url> [-CertificateFilePath <certificate_path> | [ -CertificateStoreName <certificate_store_name> -CertificateStoreLocation <certificate_store_location> [-CertificateSubjectName <certificate_subject_name> | -CertificateFingerprint <certificate_fingerprint>]]] [options]

Signs a NuGet package.

argument:

package - Path to the package(s) that needs to be signed.

options:

-OutputDirectory - Directory where the signed package should be saved. By default the original package is overwritten by the signed package.

-CertificateFilePath - File path to the certificate to be used while signing the package.

-CertificateStoreName - Name of the X.509 certificate store to use to search for the certificate. Defaults to "My", the X.509 certificate store for personal certificates.
This option should be used when specifying the certificate via -CertificateSubjectName or -CertificateFingerprint options.

-CertificateStoreLocation - Name of the X.509 certificate store use to search for the certificate. Defaults to "CurrentUser", the X.509 certificate store used by the current user.
This option should be used when specifying the certificate via -CertificateSubjectName or -CertificateFingerprint options.

-CertificateSubjectName - Subject name of the certificate used to search a local certificate store for the certificate. 
The search is a case-insensitive string comparison using the supplied value, which will find all certificates with the subject name containing that string, regardless of other subject values.
The certificate store can be specified by -CertificateStoreName and -CertificateStoreLocation options.

-CertificateFingerprint - SHA-1 fingerprint of the certificate used to search a local certificate store for the certificate.
The certificate store can be specified by -CertificateStoreName and -CertificateStoreLocation options.

-CertificatePassword - Password for the certificate, if needed.
This option can be used to specify the password for the certificate. If no password is provided, the user may be prompted for a password at run time, unless the -NonInteractive  option is passed.

-Timestamper - URL to an RFC 3161 timestamp server.

-TimestampHashAlgorithm - Hash algorithm to be used by the RFC 3161 timestamp server. Defaults to SHA256.

-HashAlgorithm - Hash algorithm to be used while generating the package manifest file. Defaults to SHA256.

-Overwrite - Switch to indicate if the current signature should be overwritten. By default the command will fail if the package already has a signature.
 
-NonInteractive - Do not prompt for user input or confirmations.

```

### Return Value

Sign Command returns one of the following exit codes when it terminates.

| Exit code     | Description |
| ------------- | ------------- |
| 0  | Execution was successful.|
| 1  | Execution has failed. |

The errors and warnings will be displayed on the console.

### Details about options
* The users should use 1 of the 2 following ways to specify the certificate to be used to sign the package - 
    1. `-CertificateFilePath` -  
The `CertificateFilePath` option is used to uniquely identify a certificate. The parameter accepts a relative or absolute file path.
    2. `-CertificateSubjectName | -CertificateFingerprint` -  
`CertificateSubjectName` and `CertificateFingerprint` options can be used to search the local certificate store. While searching the local store, if more than one matching certificates are found then we should fail and ask the user  to provide a more specific filter. Users can also use the `-CertificateStoreName` and `-CertificateStoreLocation` options to specify the certificate store name and location to be used to search for the certificate.
* In all the cases we should display the certificate being used.  
* Certificates should be displayed in the following format - 
```
    Issued to: Ankit Mishra
    Issued by: Issuing Authority
    Expires:   Sat Jul 14 14:09:34 2018
    SHA1 hash: cert hash
    Enhanced Key Usage: Usage
```

* The users should use the following way to specify the private key to be used to sign the package - 
    1. Users should provide a certificate which contains a private key, in such a case we will use that to sign the package.

### Acceptable Certificate sources
The command will support for the following certificates sources - 
 1. Certificate file - Path to the certificate file on the local file system or a network share.
 2. Certificate store- `CertificateSubjectName` and `CertificateFingerprint` options can be used to search the local certificate store. Users can also use the `-CertificateStoreName` and `-CertificateStoreLocation` options to specify the certificate store name and location to be used to search for the certificate.

### Signing Atomicity
The sign command should be atomic in nature i.e. If the command fails then the original package should not be modified. Current approach - 
 Create the manifest and signature files out of package and write into the package once all the files are ready.

### Validation on Sign
The sign command should perform the following validations before attempting to sign the package - 
 1. Validate that the package exists on disk and the process has Read/Write access to the package.
 2. If `-OutputDirectory` if passed, validate that the process has write access to the path.
 3. Validate that the user has supplied a single certificate through all of the options.
 4. If the certificate is password protected, validate that the user supplied a password using the `-CertificatePassword` option. Or prompt the user, when possible.
 5. Validate that the resolved certificate is currently valid.
 6. Validate that the user has passed a valid timestamper url using the `-Timestamper` option.
 8. Logical validation on all the supplied options.

### Corresponding commands

In future we would like to add support for the following platforms - 

* Dotnet CLI - `dotnet nuget sign <package_path> [Options]`

* MSBuild target - `msbuild /t:nugetsign <package_path> [Options]`

## Open Questions 

 **1.** Whats the best way to pass the certificate password to the sign command?  
    Currently the spec only accepts a commandline switch with clear text password. But we can also support having an 
    encrypted password stored in a nuget.config file.  
    _We will allow users to pass a clear text via commandline or prompt them (On desktop and while not in NonInteractive mode) to input it in secure string format during runtime._

 **2.** What kind of validation will we do before signing?  
    We should spec out all the validations that will be done before signing.  
    _Added to the spec_

 **3.** Do we need to add retry mechanism for the timestamper service?  
  Number of retries to get a timestamp for the signature.  
  Delay (in seconds) between the retries to get a timestamp for the signature  
  _No. Looked closely at singtool and realized that they do not have a mechanism for retry._

## Feedback
Please use the [tracking issue](https://github.com/NuGet/Home/issues/5907) to provide feedback or any questions that you might have. Thanks!
***
