Status: **Reviewing**

## Issue

The discussion around this spec is tracked here - **Repository Signatures and Countersignatures Technical Specification [#6378](https://github.com/NuGet/Home/issues/6378)** 


# NuGet Repository Signatures and Countersignatures Technical Specification

## Introduction
This specification updates the [NuGet Package Signatures Technical Specification](https://github.com/NuGet/Home/wiki/Package-Signatures-Technical-Details) by defining repository signatures and repository countersignatures. 

## Conformance keywords
The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119) and [RFC 8174](https://tools.ietf.org/html/rfc8174).

## Abbreviations
* ASN.1:  Abstract Syntax Notation.One [[ITU-T](https://www.itu.int/en/ITU-T/asn1/Pages/introduction.aspx)]
* CMS:  cryptographic message syntax [[RFC 5652](https://tools.ietf.org/html/rfc5652)]
* DER:  Distinguished Encoding Rules [[X.690](https://www.itu.int/itu-t/recommendations/rec.aspx?rec=x.690)]
* OID:  object identifier [[X.660](http://www.itu.int/rec/T-REC-X.660-201107-I/en)]

## Requirements
The general requirements are:
1. A signed package MAY contain either a repository signature or a repository countersignature.
1. A signed package MUST NOT contain both a repository signature and a repository countersignature.
1. A signed package MUST NOT contain more than 1 repository countersignature.
1. If a repository signature is present, it MUST be the primary signature.
1. If a repository countersignature is present, it MUST be a countersignature of the primary signature.
1. Repository signatures and repository countersignatures MUST contain the signing repository's V3 service index URL.

## Repository signatures and countersignatures
If a repository signature exists, it MUST be the primary signature.

A repository signature or repository countersignature MAY satisfy the requirements of any CAdES [[RFC 5126](https://tools.ietf.org/html/rfc5126)] signature but MUST satisfy CAdES-BES [[RFC 5126](https://tools.ietf.org/html/rfc5126#section-4.3.1)] requirements with the following additional requirements:
* The `commitment-type-indication` attribute [[RFC 5126](https://tools.ietf.org/html/rfc5126#section-5.11.1)] MUST be present.  The attribute MUST include the `id-cti-ets-proofOfReceipt` commitment type.
* The `signing-certificate-v2` attribute [[RFC 5126](https://tools.ietf.org/html/rfc5126#section-5.7.3.2)] MUST be present.  The hash algorithm used in this attribute MUST be a [supported hash algorithm](https://github.com/NuGet/Home/wiki/Package-Signatures-Technical-Details#supported-algorithms).
* The `signing-time` attribute [[RFC 5652](https://tools.ietf.org/html/rfc5652#section-11.3)] MUST be present.

### Repository Metadata
#### The `nuget-v3-service-index-url` attribute
The following OID identifies the `nuget-v3-service-index-url` attribute:

    1.3.6.1.4.1.311.84.2.1.1.1

Repository signatures and repository countersignatures MUST have this attribute.

This attribute MUST be a signed attribute; it MUST NOT be an unsigned attribute.  A CMS `SignerInfo` MUST NOT include multiple instances of this attribute.  This attribute MUST have exactly one attribute value.  The attribute value has the ASN.1 type `NuGetV3ServiceIndexUrl`:

```
NuGetV3ServiceIndexUrl ::= IA5String
```

The attribute value MUST be the DER-encoded, official repository V3 service index URL.  The URL MUST be a syntactically valid HTTPS URL per [[RFC 7230](https://tools.ietf.org/html/rfc7230#section-2.7.2)].

#### The `nuget-package-owners` attribute
The following OID identifies the `nuget-package-owners` attribute:

    1.3.6.1.4.1.311.84.2.1.1.2

Repository signatures and repository countersignatures MAY have this attribute but MUST NOT have the attribute if there are no package owners to include.  At least one package owner MUST be included in the attribute value.

This attribute MUST be a signed attribute; it MUST NOT be an unsigned attribute.  A CMS `SignerInfo` MUST NOT include multiple instances of this attribute.  This attribute MUST have exactly one attribute value.  The attribute value has the ASN.1 type `NuGetPackageOwners`:

```
NuGetPackageOwners ::= SEQUENCE SIZE (1..MAX) OF NuGetPackageOwner

NuGetPackageOwner ::= UTF8String (SIZE (1..MAX))
```

The attribute value MUST be DER encoded.  Individual package owner strings MUST NOT be null, an empty string, or whitespace only (e.g.:  using [`System.String.IsNullOrWhiteSpace(...)`](https://msdn.microsoft.com/en-us/library/system.string.isnullorwhitespace(v=vs.110).aspx)).

## Certificate minimum requirements
A NuGet repository signing/countersigning certificate is a NuGet package signing certificate and MUST satisfy minimum requirements described [here](https://github.com/NuGet/Home/wiki/Package-Signatures-Technical-Details#-certificate-minimum-requirements).


## References

* [NuGet Package Signatures Technical Specification](https://github.com/NuGet/Home/wiki/Package-Signatures-Technical-Details)
* [RFC 2119](https://tools.ietf.org/html/rfc2119):  Key words for use in RFCs to Indicate Requirement Levels
* [RFC 5652](https://tools.ietf.org/html/rfc5652):  Cryptographic Message Syntax (CMS)
* [RFC 7230](https://tools.ietf.org/html/rfc7230):  Hypertext Transfer Protocol (HTTP/1.1): Message Syntax and Routing
* [RFC 8174](https://tools.ietf.org/html/rfc8174):  Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words
* [X.660](http://www.itu.int/rec/T-REC-X.660-201107-I/en):  Information technology – Procedures for the operation of object identifier registration authorities: General procedures and top arcs of the international object identifier tree
* [X.690](https://www.itu.int/itu-t/recommendations/rec.aspx?rec=x.690):  Information technology – ASN.1 encoding rules: Specification of Basic Encoding Rules (BER), Canonical Encoding Rules (CER) and Distinguished Encoding Rules (DER)