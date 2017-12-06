This document contains a list of all warnings and errors that may occur during signing, verifying and using signed packages.

Package signing related errors and warnings should be in the following range -

| Log Message Type | Starting Code | Ending Code |
|------------------|---------------|-------------|
| Errors           | NU3000        | NU3099      |
| Warnings         | NU3500        | NU3599      |

# Errors

### 3000

#### Issue
Default signature issue


### NU3001

#### Issue
Input error in sign/verify command -
In sign command - 
1. The certificate file is not found.
2. The certificate file is not a valid pfx file.

In verify command -
1. Package signature is invalid.
2. Package is not signed.


### NU3002

#### Issue
Package verification fails due to one of the following - 
1. Package integrity check failed. The package has been tampered.
2. Author signature verification failed. <Exception Text>
3. Signature does not have a certificate.
4. Certificate does not meet the public key requirements.
5. Unable to validate signer certificate chain.

### NU3003

#### Issue
Invalid number of matching certificates in sign command - 
1. Multiple certificates were found that meet all the given criteria. Use the '-CertificateFingerprint' option with the hash of the desired certificate.
2. No certificates were found that meet all the given criteria. For a list of accepted ways to provide a certificate, please visit https://docs.nuget.org/docs/reference/command-line-reference



### NU3011

#### Issue
Certificate chain cannot be built for the following cases - 
1. The timestamp service's certificate chain could not be built for the following certificate - <Certificate Details>


### NU3012

#### Issue
Certificate not valid in the following cases - 
1. Author certificate was not valid when it was timestamped.


### NU3013

#### Issue
The certificate's private key cannot be read -
The following certificate cannot be used for package signing as the private key provider is unsupported.


### NU3014

#### Issue
Invalid password was provided for the certificate file '<cert_file_path>'. Please provide a valid password using the '-CertificatePassword' option


### NU3021

#### Issue
Timestamp authority response not valid in the following cases - 
1. Timestamp service's response does not meet the NuGet package signature specification: Timestamp response does not contain a matching response. 
2. Timestamp service's response does not meet the NuGet package signature specification: Timestamp response does not contain an acceptable hash algorithm.
3. Timestamp service's response does not meet the NuGet package signature specification: Timestamp signature contains invalid content type.
4. Timestamp service's response does not meet the NuGet package signature specification: Timestamp response contains invalid signature value hash.
5. Timestamp service's response does not meet the NuGet package signature specification: Timestamp service's certificate does not contain a valid Enhanced Key Usage for timestamping.


### NU3022

#### Issue
Signed package contains an invalid timestamp - 
1. The signature contains an invalid timestamp. 
   Detailed log contains more detailed failure.

<br/>
# Warnings 

### NU3500

#### Issue
Default signing warning


### NU3501

#### Issue
Certificate does not build to a trusted root - 
Signing certificate does not chain to a trusted root.


### NU3502

#### Issue
Signature information unavailable. [Currently not thrown]


### NU3521

#### Issue
No `-Timestamper` option was provided the signed package will not be timestamped. To learn more about this option, please visit https://docs.nuget.org/docs/reference/command-line-reference

