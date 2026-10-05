%%%
title = "IDN Variants for the RESTful Provisioning Protocol (RPP)"
abbrev = "IDN Variants extension for RPP"
area = "Internet"
workgroup = "Network Working Group"
submissiontype = "IETF"
keyword = [""]
TocDepth = 4
date = 2026-11-14

[seriesInfo]
name = "Internet-Draft"
value = "draft-wullink-rpp-extension-idn-variants-00"
stream = "IETF"
status = "standard"

[[author]]
initials="M."
surname="Wullink"
fullname="Maarten Wullink"
abbrev = ""
organization = "SIDN Labs"
  [author.address]
  email = "maarten.wullink@sidn.nl"
  uri = "https://sidn.nl/"

[[author]]
initials="P."
surname="Kowalik"
fullname="Pawel Kowalik"
abbrev = ""
organization = "DENIC"
  [author.address]
  email = "pawel.kowalik@denic.de"
  uri = "https://denic.de/"

%%%

.# Abstract

This document specifies an extension to the RESTful Provisioning Protocol (RPP) that addresses the handling of Internationalized Domain Name (IDN) variants. It defines the necessary data objects, operations, and registration requirements to support IDN variants within the RPP framework.

{mainmatter}

# Introduction

This extension uses the guidelines for extending the RESTful Provisioning Protocol (RPP) as described in [@!I-D.wullink-rpp-extension-guidelines].

Internationalized Domain Names (IDN) are domain names that include one or more non-ASCII characters. IDNs are represented in the Domain Name System (DNS) using ASCII Compatible Encoding (ACE), commonly known as Punycode. This allows the DNS to handle Unicode domain names while maintaining compatibility with the existing ASCII-based infrastructure.

An IDN Variant Label is an alternate label that a Label Generation Ruleset (LGR) considers equivalent to another label for registration purposes, typically because the two labels are visually, phonetically, or otherwise linguistically related within a script or language. Two or more domain names are variants of one another when the server, applying the applicable LGR, determines that their labels stand in this relationship.

A bundle is a logical grouping of two or more domain names that the server has determined to be IDN Variant Labels of one another and that share a common registrant. Each domain name in a bundle remains its own independent Domain Name Data Object, with its own lifecycle, `name`, and other data elements. A bundle is not itself a Data Object: it has no independent lifecycle and is not directly created, read, updated, or deleted through a dedicated resource or operation. Bundle membership is derived by the server as a consequence of variant detection, and a bundle MAY grow to contain more than two members as additional variants are detected over time.

RPP continues to model every request as acting on a single domain name, addressed by its own `name`; RPP does not define a request or response structure that addresses a bundle as a whole. However, an RPP specification or extension MAY define specific operations (for example, delete or transfer) whose effect, once performed on one domain name, is applied by the server to every domain name in the same bundle. Such operations MUST document this cascading behaviour explicitly; absent such a definition, an operation MUST be understood to affect only the specified domain name.

**TODO:** What operations operate on all variants in a bundle? Transfer?

# Terminology

In this document the following terminology is used.

Label Generation Ruleset (LGR) - A set of rules defining the valid labels for a registry, including the permitted repertoire, contextual rules, and, where applicable, variant mappings. Also historically known as IDN tables.

IDN Variant Label - An alternate label that an LGR considers equivalent to another label for registration purposes.

Bundle - A logical grouping of two or more domain names that the server has determined to be IDN Variant Labels of one another and that share a common registrant.

A-label - The ASCII Compatible Encoding (ACE) form of an internationalized label, as defined in Section 2.3.2.1 of [@!RFC5890].

U-label - The Unicode form of an internationalized label, as defined in Section 2.3.2.1 of [@!RFC5890].

# Conventions Used in This Document

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT","SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [@!RFC2119].

In examples, indentation and white space are provided only to illustrate element relationships and are not REQUIRED features of the protocol.

# Extension Overview

| Field | Value |
|---|---|
| Name | RPP IDN Variants |
| Identifier | urn:ietf:params:rpp:extension:idn-variants |
| Version | 1.0 |
| RPP version | 1.0 |
| Schema `$id` | https://rpp.example/schemas/ext-idn-variants.json |
| Changed base objects | domainName |
| New objects | domainVariant, domainVariants |
Table: Extension Overview
{#tbl-extension-overview}

# External Data Types

None.

# Result Codes

None.

# Problem Details

None.

# HTTP Headers

None.

# Discovery Document

None.

# Updated Data Objects

## Domain Name Data Object

* Identifier: domainName

### Data Elements

The following data elements are added to the Domain Name Data Object. The existing data element `name` is unchanged, with the following additional constraint.

* Variants
  * Identifier: variants
  * Cardinality: 0-1
  * Mutability: read-only
  * Data Type: Domain Name Variants Object
  * Direct Access: true
  * Description: The known Internationalized Domain Name (IDN) variants of the domain name, as determined by the Label Generation Ruleset (LGR).
  * Constraints: The server MAY choose not to include this data element if IDN is not supported.

### Operations

#### Create Operation

The server MUST validate the name using the procedure described in section 4.2 of [@!RFC5891]. If the validation of the IDN name failed because it contained a code point not available in the specified LGR, the server MUST return an error indicating the invalid code point. If the name does not map to the provided Unicode Name (uName), the server MUST respond with an appropriate error indicating the mismatch.

##### IDN Variant Detection

When a create request is submitted for a name associated with a Label Generation Ruleset (LGR) (`lgr`), the server MUST use LGR identified by `lgr`, to determine whether the requested name is an IDN variant of any domain name already registered in the same namespace.

* If the requested name is not a variant of any existing domain name, the server MUST proceed to create it as a new, independent Domain Name Data Object.
* If the requested name is a variant of an existing domain name, the server MUST create the requested domain name only when its registrant is the same as the registrant of the existing variant domain name. If the registrants differ, the server MUST reject the create operation and return an appropriate error.

A domain name created under this rule is provisioned and managed as its own independent Domain Name Data Object instance, addressable and operable via its own `name` (Unique Identifier), like any other Domain Name Data Object.

If the requested name is a variant of an existing domain name, the server MUST add the newly created domain name to a bundle together with the existing variant domain name(s) it was detected against. A bundle is a logical grouping of two or more domain names that the server has determined to be IDN Variant Labels of one another and that share a common registrant. A bundle is not itself a Data Object: it has no independent lifecycle and is not directly created, read, updated, or deleted through a dedicated resource or operation.

If the existing domain name detected as a variant is already a member of a bundle, the newly created domain name MUST be added to that same bundle rather than forming a new, separate bundle; a bundle MAY therefore contain more than two members.

# New Data Objects

## Domain Name Variant Object

* Name: Domain Name Variant Object
* Identifier: domainVariant
* Description: A container for a variant for an Internationalized Domain Name (IDN), can represent both registered and potential variants according to the relevant LGR.
* Data Elements:
  * Name
    * Identifier: name
    * Cardinality: 1
    * Mutability: read-only
    * Data Type: Fully Qualified Domain Name
    * Direct Access: false
    * Description: The (ACE) A-label form of the name of the variant.
    * Constraints: If the name is represented as a valid ASCII name then the name does not have to be an ACE encoded ("xn--" prefix) string.
  * Unicode Name
    * Identifier: uName
    * Cardinality: 1
    * Mutability: read-only
    * Data Type: String
    * Direct Access: false
    * Description: The Unicode representation of the "name" of the variant.
    * Constraints: (None)
  * Status
    * Identifier: status
    * Cardinality: 1
    * Mutability: read-only
    * Data Type: String
    * Direct Access: false
    * Description: The status of the variant.
    * Constraints: MUST be one of the following values: "registered", "available".

## Domain Name Variants Object

* Name: Domain Name Variants Object
* Identifier: domainVariants
* Description: A container for the variants for an Internationalized Domain Name (IDN), can represent both registered and potential variants according to the relevant LGR.
* Data Elements:
  * Label Generation Ruleset (LGR)
    * Identifier: lgr
    * Cardinality: 1
    * Data Type: String
    * Description: The Label Generation Ruleset (LGR) used to compute the variants for the Internationalized Domain Name (IDN).
    * Constraints: (None)
  * Variants
    * Identifier: variants
    * Cardinality: 0+
    * Mutability: read-only
    * Data Type: Domain Name Variant Object
    * Direct Access: false
    * Description: The Internationalized Domain Name (IDN) variants of the Domain Name Variants Object.
    * Constraints: (None)

### Operations

The Domain Name Variants Object only supports the Read operation.

#### Read Operation

* Identifier: read

The Read operation allows a client to retrieve the known Internationalized Domain Name (IDN) variants based on the provided input domain name.

* Authorisation:
  * Same authorisation rules as the Read Operation of the owning Domain Name Data Object.

* Input: Object Identifier of the owning Domain Name Data Object
* Output: Domain Variants Object

The following transient data elements are defined for this operation:

* Label Generation Ruleset (LGR)
  * Identifier: lgr
  * Cardinality: 0-1
  * Data Type: String
  * Description: Restricts the variants returned to those computed using a single named Label Generation Ruleset (LGR).
  * Constraints:
    * If present, the value MUST be a valid LGR name registered in the IANA Label Generation Rulesets registry [@IDN-Tables], and MUST be applicable to the owning domain name.
    * If absent, the server MUST compute variants using the default LGR applicable to the relevant owning TLD.
    * If the value does not identify an LGR applicable to the owning domain name, the server MUST return an appropriate error.

# Endpoints

The Domain Name Variants Object is exposed as a Direct Access sub-resource of a Domain Name resource, at the path derived by the rules in [@!I-D.ietf-rpp-core], and supports only the `"read"` operation defined for it.

| Operation | HTTP Method | URL path |
|---|---|---|
| Domain Variants: read | `"GET"` | `"/domainNames/{id}/variants"` |

A server MAY choose not to implement IDN functionality and not provide the Domain Name Variants Object endpoint, in which case it MUST return 404 Not Found or 501 Not Implemented.

## HTTP Query Parameters

The transient data element `lgr` of the Read operation of the Domain Name Variants Object MUST be conveyed, when present, as an HTTP query parameter of the same name on the request URL:

`"GET /domainNames/{id}/variants?lgr={idnLgrName}"`

Valid IDN Label Generation Ruleset (LGR) name values can be discovered by the client, using the `idn_tables` property of the Discovery document. If the `"lgr"` query parameter is omitted, the server MUST compute variants using the default LGR applicable to the relevant owning TLD. If the supplied value does not identify an LGR name applicable to the domain name, the server MUST reject the request with an appropriate error response.

**TODO:** Use either `idn_tables` or `lgr` consistently. The "default LGR" is not defined in the Discovery document, decide whether it should be.

## Internationalized Domain Names in URLs

When an Internationalized Domain Name (IDN) is used to identify a Domain Name Data Object or Host Data Object instance in a URL, in particular as the `{id}` path segment, the ASCII Compatible Encoding (ACE) A-label form of the name, as defined in [@!RFC5890], MUST be used. This requirement ensures that the resulting URL remains a valid URI as defined in [@!RFC3986], since the A-label form is composed exclusively of ASCII characters and therefore requires no percent-encoding or additional Unicode normalization when used as a URL path segment.

The Unicode (U-label) form of an internationalized name MUST NOT be used to address a resource in a URL. A client MAY submit or receive the U-label form as a separate data element as part of a resource representation, but this Unicode representation of the name as a whole MUST NOT be used when constructing or matching a URL.

This is consistent with the use of IDN in the DNS, the actual DNS name is naturally represented using its A-label form. This also avoids ambiguity: the URL identifies the DNS domain name rather than a particular Unicode representation of it.

For example, to address the domain name whose U-label is `"bücher.example"`, a client MUST use the corresponding A-label, `"xn--bcher-kva.example"`, when constructing the `{id}` path segment:

```
GET /domainNames/xn--bcher-kva.example
```

**TODO:** The restriction that the U-label MUST NOT be used in a URL is under discussion, as it conflicts with the intent of allowing access to a domain resource by both its A-label and U-label form.

# JSON Schema

The extension defines a single JSON Schema document. The document declares, using `rpp:extends`, that it extends the create and read definitions of the Domain Name Data Object, and contributes the new properties in separate definitions. It also defines the Domain Name Variant Object and the Domain Name Variants Object.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://rpp.example/schemas/ext-idn-variants.json",

  "rpp:extends": {
    "https://rpp.example/rpp/schema.json#/$defs/domainObject.read": "#/$defs/ext.domain.read"
  },

  "$defs": {
    "ext.domain.read": {
      "type": "object",
      "properties": {
        "variants": { "$ref": "#/$defs/domainVariants", "readOnly": true }
      }
    },
    "domainVariant": {
      "type": "object",
      "properties": {
        "@type": { "type": "string", "const": "domainVariant", "readOnly": true },
        "name": { "type": "string", "readOnly": true },
        "uName": { "type": "string", "readOnly": true },
        "status": {
          "type": "string",
          "enum": ["registered", "available"],
          "readOnly": true
        }
      },
      "required": ["@type", "name", "uName", "status"]
    },
    "domainVariants": {
      "type": "object",
      "properties": {
        "@type": { "type": "string", "const": "domainVariants", "readOnly": true },
        "lgr": { "type": "string", "readOnly": true },
        "variants": {
          "type": "array",
          "items": { "$ref": "#/$defs/domainVariant" },
          "readOnly": true
        }
      },
      "required": ["@type", "lgr", "variants"]
    }
  }
}
```

The following constraints cannot be expressed in JSON Schema and MUST be enforced by implementations:

- `name` MUST use the ASCII Compatible Encoding (ACE) A-label form when the domain name is internationalized, as defined in [@!RFC5890].
- `uName`, when present, MUST be normalized to Unicode Normalization Form C (NFC) as defined in [@UNICODE.NFC], and MUST convert to the exact value of `name` using the procedure described in Section 4.4 of [@!RFC5891].
- `lgr`, when present, MUST identify a valid LGR registered in the [@IDN-Tables] registry, and MUST be present whenever `uName` is present.


# Examples

Example read response for the internationalized domain name "xn--bcher-kva.example", which is already registered. The response contains the registration details along with its variants:

```json
{
  "@type": "domainName",
  "name": "xn--bcher-kva.example",
  "uName": "bücher.example",
  "lgr": "latn-1.0",
  "provMetadata": {
    "@type": "provMetadata",
    "repositoryId": "BUCHER1-REP",
    "spClientId": "ClientX",
    "crClientId": "ClientX",
    "crDate": "1999-04-03T22:00:00.0Z"
  },
  "variants": {
    "@type": "domainVariants",
    "lgr": "latn-1.0",
    "variants": [
      {
        "@type": "domainVariant",
        "name": "buecher.example",
        "uName": "buecher.example",
        "status": "registered"
      },
      {
        "@type": "domainVariant",
        "name": "bucher.example",
        "uName": "bucher.example",
        "status": "available"
      }
    ]
  }
}
```

Example response of the Read operation of the Domain Name Variants Object, `GET /domainNames/xn--bcher-kva.example/variants?lgr=latn-1.0`:

```json
{
  "@type": "domainVariants",
  "lgr": "latn-1.0",
  "variants": [
    {
      "@type": "domainVariant",
      "name": "buecher.example",
      "uName": "buecher.example",
      "status": "registered"
    },
    {
      "@type": "domainVariant",
      "name": "bucher.example",
      "uName": "bucher.example",
      "status": "available"
    }
  ]
}
```

**TODO:** Define how a request for the variants of a domain name that does not exist is handled. Normally this results in 404 Not Found.

# IANA Considerations

This document requests the registrations in this section, as described in [@!I-D.wullink-rpp-extension-guidelines].

## RPP Extensions Registry

IANA is requested to register the following extension in the "RPP Extensions" registry defined in [@!I-D.ietf-rpp-core]:

* Name: RPP IDN Variants
* Version: 1.0
* Identifier: urn:ietf:params:rpp:extension:idn-variants
* Specification URL: [This-ID]
* Description: Extension for handling Internationalized Domain Name (IDN) variants

## RPP URN Sub-namespace

IANA is requested to register the identifier `urn:ietf:params:rpp:extension:idn-variants` in the IETF URN sub-namespace `urn:ietf:params:rpp`, as defined in [@!I-D.ietf-rpp-core].

## RPP Data Object Registry

IANA is requested to register the following objects, data elements and operations in the "RPP Data Object Registry" defined in [@!I-D.ietf-rpp-data-objects]. The registration policy is "Specification Required".

### Updated Object: domainName

The following data element is added to the existing Object definition of `domainName`. The entry references this specification, so that implementations can distinguish it from the data elements of the base specification.

Object: domainName

Reference: [This-ID]

Data Elements
| Identifier | Name     | Card. | Mutability | Data Type                    | Description                                                                                                           |
| ---------- | -------- | ----- | ---------- | ---------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| variants   | Variants | 0-1   | read-only  | Domain Name Variants Object  | The known Internationalized Domain Name (IDN) variants of the domain name, as determined by the Label Generation Ruleset (LGR). |

### New Object: domainVariant

Object: domainVariant

Object Name: Domain Name Variant Object

Object Type: Component

Description: A container for a variant of an Internationalized Domain Name (IDN), can represent both registered and potential variants according to the relevant LGR.

Reference: [This-ID]

Data Elements
| Identifier | Name         | Card. | Mutability | Data Type                | Description                                                |
| ---------- | ------------ | ----- | ---------- | ------------------------ | ---------------------------------------------------------- |
| name       | Name         | 1     | read-only  | Fully Qualified Domain Name | The (ACE) A-label form of the name of the variant.      |
| uName      | Unicode Name | 1     | read-only  | String                   | The Unicode representation of the "name" of the variant.   |
| status     | Status       | 1     | read-only  | String                   | The status of the variant, either "registered" or "available". |

### New Object: domainVariants

Object: domainVariants

Object Name: Domain Name Variants Object

Object Type: Resource

Description: A container for the variants of an Internationalized Domain Name (IDN), can represent both registered and potential variants according to the relevant LGR.

Reference: [This-ID]

Data Elements
| Identifier | Name                           | Card. | Mutability | Data Type                  | Description                                                                                          |
| ---------- | ------------------------------ | ----- | ---------- | -------------------------- | ---------------------------------------------------------------------------------------------------- |
| lgr        | Label Generation Ruleset (LGR) | 1     | read-only  | String                     | The Label Generation Ruleset (LGR) used to compute the variants for the Internationalized Domain Name (IDN). |
| variants   | Variants                       | 0+    | read-only  | Domain Name Variant Object | The Internationalized Domain Name (IDN) variants of the Domain Name Variants Object.                 |

Operations

Operation: Read

Operation Identifier: read

Description: Retrieves the known Internationalized Domain Name (IDN) variants of the owning Domain Name resource.

Parameters
| Identifier | Name                           | Card. | Data Type | Description                                                                                  |
| ---------- | ------------------------------ | ----- | --------- | -------------------------------------------------------------------------------------------- |
| lgr        | Label Generation Ruleset (LGR) | 0-1   | String    | Restricts the variants returned to those computed using a single named Label Generation Ruleset (LGR). |

# Security Considerations

**TODO**

# Internationalization Considerations

**TODO**

# Privacy Considerations

**TODO**

# Change History

**TODO**

{backmatter}

{numbered="false"}
# Acknowledgements

**TODO**

<reference anchor="UNICODE.NFC" target="https://www.unicode.org/reports/tr15/">
  <front>
    <title>Unicode Normalization Forms</title>
    <author>
      <organization>Unicode Consortium</organization>
    </author>
    <date year="2026" month="08"/>
  </front>
</reference>

<reference anchor="IDN-Tables" target="https://www.iana.org/assignments/idn-tables">
  <front>
    <title>Repository of IDN Practices</title>
    <author>
      <organization>Internet Assigned Numbers Authority (IANA)</organization>
    </author>
  </front>
</reference>

<reference anchor="I-D.wullink-rpp-extension-guidelines">
  <front>
    <title>Guidelines for Extending the RESTful Provisioning Protocol (RPP)</title>
    <author initials="M." surname="Wullink" fullname="Maarten Wullink">
      <organization>SIDN Labs</organization>
    </author>
    <author initials="P." surname="Kowalik" fullname="Pawel Kowalik">
      <organization>DENIC</organization>
    </author>
    <date year="2026"/>
  </front>
  <seriesInfo name="Internet-Draft" value="draft-wullink-rpp-extension-guidelines-00"/>
</reference>
