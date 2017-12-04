Status: **In review**

# NuGet Package Signatures Technical Specification

## Introduction
This specification defines a standard for signing NuGet packages, describes how package signatures are embedded inside the NuGet package to which they apply, and how package signatures are generated and validated. 

## Conformance Keywords
The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119) and [RFC 8174](https://tools.ietf.org/html/rfc8174).

## Definitions
* Package entry:  a file within a NuGet package without consideration for the file's file type.  A NuGet package is a ZIP file, and a ZIP file may contain a file with a file type that is not a regular file (e.g.:  a directory, symbolic link, etc.).  This specification will use "package entry" (singular) and "package entries" (plural) when necessary to avoid file type confusion when referencing files within a package.
* Package reader:  anything that attempts to read input as a NuGet package.
* Package writer:  anything that attempts to create or modify a NuGet package.

## Abbreviations
* CMS:  cryptographic message syntax [[RFC 5652](https://tools.ietf.org/html/rfc5652)]
* EKU:  extended key usage [[RFC 5280](https://tools.ietf.org/html/rfc5280)]
* OID:  object identifier [[X.660](http://www.itu.int/rec/T-REC-X.660-201107-I/en)]

## Requirements
The general requirements are:
1. A signed package MUST be consumable by package readers and writers that do not support package signing.
1. A package signature MUST be embedded in a package file.
1. A signed package MUST have exactly 1 primary signature.  (The root CMS `SignedData` must have exactly 1 `SignerInfo` in its `signerInfos` collection.  Cosigning is not permitted.)
1. The primary signature SHOULD be either an author signature or a repository signature.
1. An author signature MUST be a primary signature.  (To apply an author signature on an already signed package, the existing primary signature must first be removed.)

The package signing feature roadmap consists of multiple rollout stages.  Repository signatures, which are planned after the first rollout stage, will be detailed in a later specification.  They are minimally covered in this specification to make reservations for them.  Package writers SHOULD NOT generate repository signatures until the repository signature specification has been finalized.

## Out of Scope
NuGet package signing will not support:
* 64-bit ZIP files
* ZIP files signed with other techniques

## Description of a signed NuGet package
A NuGet package is based on the widely used ZIP file format.

A signed NuGet package MUST contain a package signature file, which is described in a [later section](#PackageSignatureFile).

A package reader that does not support package signing SHOULD NOT attach special significance to this file and MAY choose to ignore it.

## <a id="DeterminationIfPackageIsSigned"></a> Determination if a package is signed
An efficient test is required to determine if a package should be validated as a signed package.  The test MUST NOT produce false negatives.  The test MAY produce false positives; however, it is important that the test minimize them.
A package SHOULD be considered signed if and only if the package has a package signature file.

Passing of this test MUST NOT imply package integrity or signature validity.

## The properties document
A properties document is a collection of properties (name-value pairs) and has the following ABNF format:
```
properties-document = header-section 1*section
header-section      = 1*property EOL
section             = 1*property EOL
EOL                 = CRLF / LF
property            = name ":" value EOL
name                = 1*namechar
namechar            = ALPHA / DIGIT / . / - / "/"
value               = 1*valuechar
```
`valuechar` is defined as any UTF-8 character excluding `NUL`, `CR`, and `LF`.  See [RFC 5234 section B.1](https://tools.ietf.org/html/rfc5234#appendix-B.1) for additional definitions.
A properties document MUST be UTF-8 encoded.

## <a id="PackageSignatureFile"></a> The package signature file
The package signature file has a package entry with the following full, case-sensitive file name (e.g.:  as returned by the System.IO.Compression.ZipArchiveEntry.FullName property):
```
.signature.p7s
```
The entry's file type MUST be a regular file (e.g.:  and not a directory, symbolic link, etc.).  The entry MUST NOT be compressed.

The entry data MUST be a single CMS `SignedData` as encoded by  [`System.Security.Cryptography.Pkcs.SignedCms.Encode()`](https://msdn.microsoft.com/en-us/library/system.security.cryptography.pkcs.signedcms.encode(v=vs.110).aspx).  The CMS content (`SignedData.encapContentInfo.eContent`) MUST be a properties document.  Property names (`name`) and values (`value`) are case-sensitive.

The properties document's `header-section` MUST contain the following property:
* `Version`:  This property defines the package signature format version.  The value MUST be `1`.

Package readers and writers compliant with this specification MUST fail a signature verification or signing operation if the version is anything other than this value.

The properties document's first `section` MUST contain the following property:
* `Hash-Algorithm-Oid-Hash`:
  * `Hash-Algorithm-Oid` MUST be the OID for a [supported hash algorithm](#SupportedAlgorithms).  The actual property name is based on the hash algorithm's OID (e.g.:  `2.16.840.1.101.3.4.2.1-Hash`, `2.16.840.1.101.3.4.2.2-Hash`, or `2.16.840.1.101.3.4.2.3-Hash`).
  * The property value MUST be a Base64-encoded hash of the entire sequence of bytes of the unsigned package using the hash algorithm specified by `Hash-Algorithm-Oid`.

An example properties document is:

```
Version:1

2.16.840.1.101.3.4.2.1-Hash:l76DnQSKN9AKihpevTkEaojKaLaZqmcmj9q76TwdZthfChmtVPLtC5ijl6ydsk1sMBMdF2Lcg8FKj6y1U4VMKg==

```
Author and repository signatures are identified through the `commitment-type-indication` attribute [[RFC 5126]( https://tools.ietf.org/html/rfc5126.html#section-5.11.1)].  Author and repository signatures MUST have this attribute as follows:
* The `id-cti-ets-proofOfOrigin` commitment type is reserved for author signatures.  An author signature MUST use this commitment type.
* The `id-cti-ets-proofOfReceipt` commitment type is reserved for repository signatures.  A repository signature MUST use this commitment type.

A signature MUST NOT specify both the author and repository commitment types.  A signature which is neither an author nor repository signature MUST NOT use the aforementioned commitment types.

An author signature MAY satisfy the requirements of any CAdES signature but MUST satisfy CAdES-BES requirements with the following additional requirements:
* The `SigningCertificateV2` attribute [[RFC 5035](https://tools.ietf.org/html/rfc5035)] MUST be present.
* The `signing-time` attribute [[RFC 5652](https://tools.ietf.org/html/rfc5652#section-11.3)] MUST be present.

An author signature SHOULD extend to CAdES-T by adding the `signature-time-stamp` attribute [[RFC 5126](https://tools.ietf.org/html/rfc5126#section-6.1.1)] to provide long-term signature validity even after the signing certificate's validity period has passed.

If a signature, primary or not, lacks a timestamp, then that signature SHOULD be considered expired and subsequently ignored if the current time according to the package reader is outside of the signing certificate's validity period.  If specifically the primary signature's signing certificate has expired and the primary signature lacks a timestamp, then a package reader MAY still use information in the signed CMS to verify package integrity but MUST otherwise treat the signature as expired and the package as unsigned.  An exception to signature expiration is that package readers MAY choose to treat timestamps as non-expiring.

The signed CMS certificates collection (`SignedData.certificates`) SHOULD contain all certificates, including the root certificate, necessary to build a single, complete, valid chain for each signature in the signed CMS object.

Package signing certificate requirements and minimum requirements for hash and signature algorithms are listed in the following sections.

## <a id="CertificateMinimumRequirements"></a> Certificate minimum requirements
A NuGet package signing certificate MUST meet the following minimum requirements:
1. The certificate MUST have the `id-kp-codeSigning` EKU per [RFC 5280 section 4.2.1.12](https://tools.ietf.org/html/rfc5280#section-4.2.1.12).
1. The certificate MUST have an RSA public key length of 2048 bits or higher.

A timestamping certificate MUST 
meet the following minimum requirements:
1. The certificate MUST have the `id-kp-timeStamping` EKU per [RFC 5280 section 4.2.1.12](https://tools.ietf.org/html/rfc5280#section-4.2.1.12).
1. The certificate MUST have an RSA public key length of 2048 bits or higher.

At signing time, a certificate MUST be within its validity period according to the package writer and MUST NOT be not revoked.  At validation time, the certificate's revocation status SHOULD be rechecked; however, it is reasonable for package readers to fail open if revocation status is unavailable (e.g.:  a CRL is inaccessible).

Certificates SHOULD NOT have the lifetime signing EKU (1.3.6.1.4.1.311.10.3.13).  Package readers and writers may not check for or enforce the lifetime signing EKU.

## <a id="SupportedAlgorithms"></a>Supported algorithms
The following hash and signature algorithms SHOULD be supported:

| Hash Algorithm  | `Hash-Algorithm-Oid`   |
| --------------- |------------------------|
| SHA-2-256       | 2.16.840.1.101.3.4.2.1 |
| SHA-2-384       | 2.16.840.1.101.3.4.2.2 |
| SHA-2-512       | 2.16.840.1.101.3.4.2.3 |

The RSA public key algorithm SHOULD be supported.

RSA-based signatures MUST use the `RSASSA-PKCS1-v1_5` padding mode [[RFC 8017](https://tools.ietf.org/html/rfc8017#section-8.2)].  This is to help ensure compatibility across all platforms (https://github.com/dotnet/corefx/blob/master/Documentation/architecture/cross-platform-cryptography.md#rsa).

Over time algorithms may be deprecated and replaced with newer, more secure algorithms.  An example is SHA-1's deprecation in favor of SHA-2.  Package readers and writers that support package signing MAY block acceptance and creation, respectively, of new packages signed with a deprecated algorithm.  Older package readers and writers that support package signing SHOULD treat packages signed with a newer, unsupported algorithm as:
* unsigned for read operations.  (Do not block installation of such a package, but also do not attempt to validate the signature.)
* signed for write operations.  (An older writer can still remove an existing signature.)

## Signing a package
The following are sample steps for author signing a package.  Unless otherwise indicated, package writers SHOULD err on the side of caution and treat unexpected failures as fatal.
1. [Determine if the package is signed](#DeterminationIfPackageIsSigned).
    1. If the package is signed and the sign operation should not overwrite an existing signature, fail the sign operation with a message indicating that the package is already signed.
    1. If the package is signed and the sign operation should overwrite an existing signature, remove the existing signature and continue with step 2.
    1. If the package is not signed, continue with step 2.
1. Verify that the signing certificate satisifies [minimum requirements](#CertificateMinimumRequirements).
1. Verify that the hash, signature, and timestamp hash algorithms are [supported](#SupportedAlgorithms).
1. Generate a package signature file.
    1. Create an author signature.
    1. Obtain a timestamp for the author signature.
    1. Verify that the timestamp signature algorithm is [supported](#SupportedAlgorithms).
    1. Extend the author signature to CAdES-T.
    1. Encode the author signature CMS `SignedData`.
    1. Write the encoded author signature to file.
1. Add the package signature file as an uncompressed entry to the package to be signed.

## Unsigning a package
If the package contains a package signature file entry, remove it.  The package is no longer signed.
 
## Validating a signed package
The following are sample steps for verifying a signed package.  Unless otherwise indicated, package readers SHOULD err on the side of caution and treat unexpected validation failures as fatal and block restoration of the package.  Example failures include:
* failure to decode the package signature file contents as a signed CMS fails
* failure to read expected ZIP structures
* failure parse a properties document
* failure to validate package integrity

Package readers MAY impose more stringent requirements during signature validation. 
In the case of signed packages downloaded by a plugin, some steps below MAY be delegated to the plugin to fully or partially implement.
1. [Determine if the package is signed](#DeterminationIfPackageIsSigned).  If it is not signed, stop further validation.
1. Verify that the package signature file entry is not compressed and has a regular file file type.
1. Verify that the package signature format is supported.
    1. Decode the contents of the package signature file entry as a signed CMS.
    1. Parse the signed CMS content as a properties document.
    1. Verify that the package signature format version as indicated by the `Version` property value is supported.
1. Verify package integrity.
    1. Verify that the hash algorithm indicated by the `Hash-Algorithm-Oid-Hash` property is supported.  If it is not supported, stop further validation and treat the package as unsigned.
    1. Compute a new hash using the same hash algorithm from step 4.1.
        1. Open a read-only binary stream for the package file.
        1. Compute a hash over the stream while accounting for package changes introduced by adding the package signature file to the package.  For example:
            1. Exclude the package signature file's local file and central directory headers from hashing.
            1. For each local file header after the package signature file entry's local file header directory header instead of hashing the value of the  central directory header field in the signed package, compute and hash the value for the unsigned package.
                * relative offset of local header
            1. In the end of central directory record, instead of hashing the values of the following fields in the signed package, compute and hash their values for the unsigned package.
                * size of the central directory
                * offset of start of central directory with respect to the starting disk number
    1. Verify that the expected and actual package hashes are identical.
1. Verify primary signature validity and trust.
    1. Verify that the signed CMS has one `SignerInfo`.
    1. Check a `signature-time-stamp` attribute on the primary signature.
        1. If the timestamp exists, continue with the next step.  Otherwise, store the local machine's current UTC time in variables `TimeStampLowerLimit` and `TimeStampUpperLimit` and continue with step 6.
        1. Verify that the timestamp hash in `TSTInfo.messageImprint` matches the hash of the signature to which the timestamp applies.
        1. Verify that the timestamp hash and signature algorithms are [supported](#SupportedAlgorithms).
        1. Verify that the timestamp certificate satisifies [minimum requirements](#CertificateMinimumRequirements).
        1. Create an additional certificate store consisting of:
            * certificates in the `SignedData.certificates` collection
            * certificates in the timestamp signature's `signing-certificate` [[RFC 2634](https://tools.ietf.org/html/rfc2634)] or `SigningCertificateV2` [[RFC 5035](https://tools.ietf.org/html/rfc5035)] attributes, if present
        1. Using the additional certificate store from step 5.2.5, build a chain for the timestamp signing certificate with the `id-kp-timeStamping` EKU.  
        1. Retrieve the timestamp's generalized time from `TSTInfo.genTime`.
        1. Retrieve the timestamp's accuracy.  If the accuracy is explicitly specified in `TSTInfo.accuracy`, use that value.  If the accuracy is not explicitly specified and `TSTInfo.policy` is the baseline time-stamp policy [[RFC 3628](https://tools.ietf.org/html/rfc3628#section-5.2)], use an accuracy of 1 second.  Otherwise, use an accuracy of 0. 
        1. Calculate the timestamp range using the lower and upper limits per [RFC 3161 section 2.4.2](https://tools.ietf.org/html/rfc3161#section-2.4.2) and store the limits in variables `TimeStampLowerLimit` and `TimeStampUpperLimit`, respectively.
1. Verify primary signature validity.
    1. Verify that the signature algorithm is [supported](#SupportedAlgorithms).
    1. Verify that the signing certificate satisifies [minimum requirements](#CertificateMinimumRequirements).
    1. Verify signature validity (e.g.:  `SignerInfo.CheckSignature(verifySignatureOnly: true)`).
    1. Verify that the time range from `TimeStampLowerLimit` to `TimeStampUpperLimit` timestamp is entirely within the certificate's validity period.  If the time range is entirely within the certificate's validity period, continue to the next step.  Otherwise, the signature is invalid and package readers MUST ignore the signature.  Package readers MAY allow subsequent use of the package as an unsigned package.  For example, if a signed package has only an author signature and this step places the time range after the certificate's validity period, then the author signature validity has expired, and the package may be still be installed but only as though the package had no signature.
    1. Verify that the signing certificate is trusted.
    1. Create an additional certificate store consisting of:
        * certificates in the `SignedData.certificates` collection
        * certificates in the primary signature's `SigningCertificateV2` [[RFC 5035](https://tools.ietf.org/html/rfc5035)] attribute, if present
    1. Using the additional certificate store from step 6.6, build a chain for the signing certificate.
1. If no failures have been encountered, treat the package as a valid signed package.

## References

* [RFC 2119](https://tools.ietf.org/html/rfc2119):  Key words for use in RFCs to Indicate Requirement Levels
* [RFC 2634](https://tools.ietf.org/html/rfc2634):  Enhanced Security Services for S/MIME
* [RFC 3161](https://tools.ietf.org/html/rfc3161):  Internet X.509 Public Key Infrastructure Time-Stamp Protocol (TSP)
* [RFC 3628](https://tools.ietf.org/html/rfc3628):  Policy Requirements for Time-Stamping Authorities (TSAs)
* [RFC 5035](https://tools.ietf.org/html/rfc5035):  Enhanced Security Services (ESS) Update: Adding CertID Algorithm Agility
* [RFC 5126](https://tools.ietf.org/html/rfc5126):  CMS Advanced Electronic Signatures (CAdES)
* [RFC 5234](https://tools.ietf.org/html/rfc5234):  Augmented BNF for Syntax Specifications: ABNF
* [RFC 5280](https://tools.ietf.org/html/rfc5280):  Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile
* [RFC 5652](https://tools.ietf.org/html/rfc5652):  Cryptographic Message Syntax (CMS)
* [RFC 8174](https://tools.ietf.org/html/rfc8174):  Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words
* [APPNOTE.TXT](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT):  ZIP File Format Specification from PKWARE, Inc., version 6.3.4 (2014)