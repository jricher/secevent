---
title: Subject Identifiers for Security Event Tokens
abbrev: secevent-subject-identifiers
docname: draft-ietf-secevent-subject-identifiers-06
date: 2020-09-04
category: std
ipr: trust200902

area: Security
workgroup: Security Events Working Group
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: A. Backman
    name: Annabelle Backman
    organization: Amazon
    email: richanna@amazon.com
    role: editor

 -
    ins: M. Scurtescu
    name: Marius Scurtescu
    organization: Coinbase
    email: marius.scurtescu@coinbase.com

normative:
  BCP26: RFC8126
  E164:
    title: The international public telecommunication numbering plan
    target: http://www.itu.int/rec/T-REC-E.164-201011-I/en
    author:
      org: International Telecommunication Union
    date: 2010
  IANA.JWT.Claims:
    title: JSON Web Token Claims
    target: http://www.iana.org/assignments/jwt
    author:
      org: IANA

informative:
  OpenID.Core:
    title: OpenID Connect Core 1.0
    target: http://openid.net/specs/openid-connect-core-1_0.html
    date: November 2014
    author:
     -
       ins: N. Sakimura
       name: Nat Sakimura
       org: Nomura Research Institute, Ltd.

     -
       ins: J. Bradley
       name: John Bradley
       org: Ping Identity

     -
       ins: M. Jones
       name: Michael B. Jones
       org: Microsoft

     -
       ins: B. de Medeiros
       name: Breno de Medeiros
       org: Google

     -
       ins: C. Mortimore
       name: Chuck Mortimore
       org: Salesforce

--- abstract

Security events communicated within Security Event Tokens may support a variety of identifiers to identify the subject and/or other principals related to the event.  This specification formalizes the notion of subject identifiers as named sets of well-defined claims describing the subject, a mechanism for representing subject identifiers within a JSON object such as a JSON Web Token (JWT) or Security Event Token (SET), and a registry for defining and allocating names for these claim sets.

--- middle

Introduction {#intro}
============
As described in Section 1.2 of SET {{!RFC8417}}, the subject of a security event may take a variety of forms, including but not limited to a JWT {{!RFC7519}} principal, an IP address, a URL, etc. Furthermore, even in the case where the subject of an event is more narrowly scoped, there may be multiple ways by which a given subject may be identified. For example, an account may be identified by an opaque identifier, an email address, a phone number, a JWT `iss` claim and `sub` claim, etc., depending on the nature and needs of the transmitter and receiver. Even within the context of a given transmitter and receiver relationship, it may be appropriate to identify different accounts in different ways, for example if some accounts only have email addresses associated with them while others only have phone numbers. Therefore it can be necessary to indicate within a SET the mechanism by which the subject of the security event is being identified.

To address this problem, this specification defines Subject Identifiers - JSON {{!RFC7159}} objects containing information identifying a subject - and Subject Identifier Types - named sets of rules describing how to encode different kinds of subject identifying information (e.g., an email address, or an issuer and subject pair) as a Subject Identifier.


Below is a non-normative example of a Subject Identifier that identifies a subject by email address, using the Email Subject Identifier Type.

~~~
{
  "subject_type": "email",
  "email": "user@example.com",
}
~~~
{: #figexampleintro title="Example: Subject Identifier using the Email Subject Identifier Type"}

Subject Identifiers are intended to be a general purpose mechanism for identifying principals within JSON objects. Below is a non-normative example of a JWT that uses a Subject Identifier in the `sub_id` claim (defined in this specification) to identify its subject.

~~~
{
  "iss": "issuer.example.com",
  "sub_id": {
    "subject_type": "phone_number",
    "phone_number": "+12065550100",
  },
}
~~~
{: #figexampleintro2 title="Example: JWT using a Subject Identifier with the sub_id claim"}

Below is a non-normative example of a SET containing a hypothetical security event describing the interception of a message, using Subject Identifiers to identify the sender, intended recipient, and interceptor.

~~~
{
  "iss": "issuer.example.com",
  "iat": 1508184845,
  "aud": "aud.example.com",
  "events": {
    "https://secevent.example.com/events/message-interception": {
      "from": {
        "subject_type": "email",
        "email": "alice@example.com",
      },
      "to": {
        "subject_type": "email",
        "email": "bob@example.com",
      },
      "interceptor": {
        "subject_type": "email",
        "email": "eve@example.com",
      },
    },
  },
}

~~~
{: #figexampleintro3 title="Example: SET with an event payload containing multiple Subject Identifiers"}

Notational Conventions {#conv}
======================
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in {{!RFC2119}}.

Definitions {#defn}
---------------
This specification utilizes terminology defined in {{!RFC7159}}, {{!RFC7519}}, and {{!RFC8417}}.

Subject Identifiers {#sub-ids}
===================
A Subject Identifier is a JSON {{!RFC7159}} object whose contents may be used to identify a principal within some context.  A Subject Identifier Type is a named definition of a set of information that may be used to identify a principal, and the rules for encoding that information as a Subject Identifier.  A Subject Identifier MUST conform to a specific Subject Identifier Type, and MUST contain a `subject_type` member whose value is the name of that Subject Identifier Type.

Every Subject Identifier Type MUST have a unique name registered in the IANA "Security Event Subject Identifier Types" registry established by {{iana-sub-id-types}}, or a Collision-Resistant Name as defined in {{!RFC7519}}.  Subject Identifier Types that are expected to be used broadly by a variety of parties SHOULD be registered in the "Security Event Subject Identifier Types" registry.

A Subject Identifier Type MAY describe more members than are strictly necessary to identify a subject, and MAY describe conditions under which those members are required, optional, or prohibited.

Aside from the `subject_type` member whose definition is given above, every member within a Subject Identifier MUST match the format specified for that member by the Subject Identifier's Subject Identifier Type. A Subject Identifier MUST NOT contain any members prohibited or not described by its Subject Identifier Type, and MUST contain all members required by its Subject Identifier Type.

Subject Identifier Types versus Principal Types
-----------------------------------------------
A Subject Identifier Type describes a way to identify a principal, but does not explicitly indicate the type of that principal (e.g., user, group, network connection, baseball team, astronomic object).  Consequently Subject Identifiers remove ambiguity around how a principal is being identified, and how to parse an identifying structure, but they do not remove ambiguity around how to resolve that identifier to a principal.  For example, consider a directory management API that allows callers to identify users and groups through both immutable unique identifiers and mutable email addresses.  Such an API could use Subject Identifiers to disambiguate between which of these two types of identifiers is in use.  However, the service would have to determine whether the principal is a user or group via some other means, such as by querying a database or by inferring the type from the API contract.

Subject Identifier Type Definitions
-----------------------------------

The following Subject Identifier Types are registered in the IANA "Security Event Subject Identifier Types" registry established by {{iana-sub-id-types}}.

### Account Subject Identifier Type {#sub-id-acct}
The Account Subject Identifier Type identifies a principal using an account at a service provider, identified with an `acct` URI as defined in {{!RFC7565}}.  Subject Identifiers of this type MUST contain a `uri` member whose value is the `acct` URI for the subject.  The `uri` member is REQUIRED and MUST NOT be null or empty.  The Account Subject Identifier Type is identified by the name `account`.

Below is a non-normative example Subject Identifier for the Account Subject Identifier Type:

~~~
{
  "subject_type": "account",
  "uri": "acct:example.user@service.example.com",
}
~~~
{: #figexamplesubidaccount title="Example: Subject Identifier for the Account Subject Identifier Type"}

### Email Subject Identifier Type {#sub-id-email}
The Email Subject Identifier Type identifies a principal using an email address.  Subject Identifiers of this type MUST contain an `email` member whose value is a string containing the email address of the principal, formatted as an `addr-spec` as defined in Section 3.4.1 of {{!RFC5322}}. The `email` member is REQUIRED and MUST NOT be null or empty. The value of the `email` member SHOULD identify a mailbox to which email may be delivered, in accordance with {{!RFC5321}}. The Email Subject Identifier Type is identified by the name `email`.

Below is a non-normative example Subject Identifier for the Email Subject Identifier Type:

~~~
{
  "subject_type": "email",
  "email": "user@example.com",
}
~~~
{: #figexamplesubidemail  title="Example: Subject Identifier for the Email Subject Identifier Type"}

#### Email Canonicalization {#email-canon}
Many email providers will treat multiple email addresses as equivalent. While the domain portion of an {{?RFC5322}} email address is consistently treated as case-insensitive per {{?RFC1034}}, some providers treat the local part of the email address as case-insensitive as well, and consider "user@example.com", "User@example.com", and "USER@example.com" as the same email address. This has led users to view these strings as equivalent, driving service providers to implement proprietary email canonicalization algorithms to ensure that email addresses entered by users resolve to the same canonical string. When receiving an Email Subject Identifier, the recipient SHOULD use their implementation's canonicalization algorithm to resolve the email address to the same string used in their system.

### Phone Number Subject Identifier Type {#sub-id-phone}
The Phone Number Subject Identifier Type identifies a principal using a telephone number.  Subject Identifiers of this type MUST contain a `phone_number` member whose value is a string containing the full telephone number of the principal, including international dialing prefix, formatted according to [E.164](#E164). The `phone_number` member is REQUIRED and MUST NOT be null or empty. The Phone Number Subject Identifier Type is identified by the name `phone_number`.

Below is a non-normative example Subject Identifier for the Email Subject Identifier Type:

~~~
{
  "subject_type": "phone_number",
  "phone_number": "+12065550100",
}
~~~
{: #figexamplesubidphone  title="Example: Subject Identifier for the Phone Number Subject Identifier Type."}

### Issuer and Subject Subject Identifier Type {#sub-id-iss-sub}
The Issuer and Subject Subject Identifier Type identifies a principal using a pair of `iss` and `sub` members, analagous to how subjects are identified using the `iss` and `sub` claims in [OpenID Connect](#OpenID.Core) ID Tokens.  These members MUST follow the formats of the `iss` member and `sub` member defined by {{!RFC7519}}, respectively.  Both the `iss` member and the `sub` member are REQUIRED and MUST NOT be null or empty. The Issuer and Subject Subject Identifier Type is identified by the name `iss_sub`.

Below is a non-normative example Subject Identifier for the Issuer and Subject Subject Identifier Type:

~~~
{
  "subject_type": "iss_sub",
  "iss": "http://issuer.example.com/",
  "sub": "145234573",
}
~~~
{: #figexamplesubidisssub  title="Example: Subject Identifier for the Issuer and Subject Subject Identifier Type"}

### Aliases Subject Identifier Type {#sub-id-aliases}
The Aliases Subject Identifier Type describes a subject that is identified with a list of different Subject Identifiers. It is intended for use when a variety of identifiers have been shared with the party that will be interpreting the Subject Identifier, and it is unknown which of those identifiers they will recognize or support.  Subject Identifiers of this type MUST contain an `identifiers` member whose value is a JSON array containing one or more Subject Identifiers.  Each Subject Identifier in the array MUST identify the same entity.  The `identifiers` member is REQUIRED and MUST NOT be null or empty.  It MAY contain multiple instances of the same Subject Identifier Type (e.g., multiple Email Subject Identifiers), but SHOULD NOT contain exact duplicates.  This type is identified by the name `aliases`. 

`alias` Subject Identifiers MUST NOT be nested; i.e., the `identifiers` member of an `alias` Subject Identifier MUST NOT contain a Subject Identifier of type `aliases`.

Below is a non-normative example Subject Identifier for the Aliases Subject Identifier Type:

~~~
{
  "subject_type": "aliases",
  "identifiers": [
    {
      "subject_type": "email",
      "email": "user@example.com",
    },
    {
      "subject_type": "phone_number",
      "phone_number": "+12065550100",
    },
    {
      "subject_type": "email",
      "email": "user+qualifier@example.com",
    }
  ],
}
~~~
{: #figexamplesubididtoken  title="Example: Subject Identifier for the Aliases Subject Identifier Type"}

Subject Identifiers in JWTs {#jwt-claims}
===========================

"sub_id" Claim {#jwt-claims-sub_id}
--------------
The `sub` JWT Claim is defined in Section 4.1.2 of {{!RFC7519}} as containing a string value, and therefore cannot contain a Subject Identifier (which is a JSON object) as its value.  This document defines the `sub_id` JWT Claim, in accordance with Section 4.2 of {{!RFC7519}}, as a common claim that identifies the subject of the JWT using a Subject Identifier.  When present, the value of this claim MUST be a Subject Identifier that identifies the principal that is the subject of the JWT.  The `sub_id` claim MAY be included in a JWT, whether or not the `sub` claim is present.  When both the `sub` and `sub_id` claims are present in a JWT, they MUST identify the same principal.

When processing a JWT with both `sub` and `sub_id` claims, implementations MUST NOT rely on both claims to determine the subject.  An implementation MAY attempt to determine the subject from one claim and fall back to using the other if it determines it does not understand the format of the first claim.  For example, an implementation may attempt to use `sub_id`, and fall back to using `sub` upon finding that `sub_id` contains a Subject Identifier whose type is not recognized by the implementation.

Below are non-normative examples of JWTs containing the `sub_id` claim:

~~~
{
  "iss": "issuer.example.com",
  "sub_id": {
    "subject_type": "email",
    "email": "user@example.com",
  },
}
~~~
{: #figexamplejwtsubidemail title="Example: JWT containing a `sub_id` claim and no `sub` claim"}

~~~
{
  "iss": "issuer.example.com",
  "sub": "user@example.com",
  "sub_id": {
    "subject_type": "email",
    "email": "user@example.com",
  },
}
~~~
{: #figexamplejwtsamesubtype title="Example: JWT where both the `sub` and `sub_id` claims identify the subject using the same identifier"}

~~~
{
  "iss": "issuer.example.com",
  "sub": "user@example.com",
  "sub_id": {
    "subject_type": "email",
    "email": "elizabeth@example.com",
  },
}
~~~
{: #figexamplejwtdiffsubvalues title="Example: JWT where both the `sub` and `sub_id` claims identify the subject using different values of the same identifier type"}

~~~
{
  "iss": "issuer.example.com",
  "sub": "user@example.com",
  "sub_id": {
    "subject_type": "account",
    "uri": "acct:example.user@service.example.com",
  },
}
~~~
{: #figexamplejwtdiffsubtype title="Example: JWT where the `sub` and `sub_id` claims identify the subject via different types of identifiers"}

"sub_id" and "iss_sub" Subject Identifiers
------------------------------------------
The `sub_id` claim MAY contain an `iss_sub` Subject Identifier.  In this case, the JWT's `iss` claim and the Subject Identifier's `iss` member MAY be different.  For example, an [OpenID Connect](#OpenID.Core) client may construct such a JWT when issuing a JWT back to its OpenID Connect Identity Provider, in order to communicate information about the services' shared subject principal using an identifier the Identity Provider is known to understand.  Similarly, the JWT's `sub` claim and the Subject Identifier's `sub` member MAY be different.  For example, this may be used by an OpenID Connect client to communicate the subject principal's local identifier at the client back to its Identity Provider.

Below are non-normative examples of a JWT where the `iss` claim and `iss` member within the `sub_id` claim are the same, and a JWT where they are different.

~~~
{
  "iss": "issuer.example.com",
  "sub_id": {
    "subject_type": "iss_sub",
    "iss": "issuer.example.com",
    "sub": "example_user",
  },
}
~~~
{: #figexamplejwtsameiss title="Example: JWT with a `iss_sub` Subject Identifier where JWT issuer and subject issuer are the same"}

~~~
{
  "iss": "client.example.com",
  "sub_id": {
    "subject_type": "iss_sub",
    "iss": "issuer.example.com",
    "sub": "example_user",
  },
}
~~~
{: #figexamplejwtdiffiss title="Example: JWT with an `iss_sub` Subject Identifier where the JWT issuer and subject issuer are different"}

~~~
{
  "iss": "client.example.com",
  "sub": "client_user",
  "sub_id": {
    "subject_type": "iss_sub",
    "iss": "issuer.example.com",
    "sub": "example_user",
  },
}
~~~
{: #figexamplejwtdiffisssub title="Example: JWT with an `iss_sub` Subject Identifier where the JWT `iss` and `sub` claims differ from the Subject Identifier's `iss` and `sub` members"}

Privacy Considerations {#privacy}
======================

Identifier Correlation
----------------------
The act of presenting two or more identifiers for a single principal together (e.g., within an `aliases` Subject Identifier, or via the `sub` and `sub_id` JWT claims) may communicate more information about the principal than was intended.  For example, the entity to which the identifiers are presented, now knows that both identifiers relate to the same principal, and may be able to correlate additional data based on that.  When transmitting Subject Identifiers, the transmitter SHOULD take care that they are only transmitting multiple identifiers together when it is known that the recipient already knows that the identifiers are related (e.g., because they were previously sent to the recipient as claims in an OpenID Connect ID Token), or when correlation is essential to the use case.

The considerations described in Section 6 of {{!RFC8417}} also apply when Subject Identifiers are used within SETs.  The considerations described in Section 12 of {{!RFC7519}} also apply when Subject Identifiers are used within JWTs.

Security Considerations {#security}
=======================

Confidentiality and Integrity
-----------------------------
This specification does not define any mechanism for ensuring the confidentiality or integrityi of a Subject Identifier.  Where such properties are required, implementations MUST use mechanisms provided by the containing format (e.g., integrity protecting SETs or JWTs using JWS {{?RFC7515}}), or at the transport layer or other layer in the application stack (e.g., using TLS {{?RFC8446}}).

Further considerations regarding confidentiality and integrity of SETs can be found in Section 5.1 of {{!RFC8417}}.  

IANA Considerations {#iana}
===================

Security Event Subject Identifier Types Registry {#iana-sub-id-types}
------------------------------------------------
This document defines Subject Identifier Types, for which IANA is asked to create and maintain a new registry titled "Security Event Subject Identifier Types".  Initial values for the Security Event Subject Identifier Types registry are given in {{sub-ids}}.  Future assignments are to be made through the Expert Review registration policy {{!BCP26}} and shall follow the template presented in {{iana-sub-id-types-template}}.

It is suggested that multiple Designated Experts be appointed who are able to represent the perspectives of different applications using this specification, in order to enable broadly informed review of registration decisions.  In cases where a registration decision could be perceived as creating a conflict of interest for a particular Expert, that Expert should defer to the judgment of the other Experts.

### Registry Location
(This section to be removed by the RFC Editor before publication as an RFC.)

The authors recommend that the Subject Identifier Types registry be located at `https://www.iana.org/assignments/secevent/`.

### Registration Template {#iana-sub-id-types-template}

{: vspace="0"}
Type Name
: The name of the Subject Identifier Type, as described in {{sub-ids}}. The name MUST be an ASCII string consisting only of lower-case characters ("a" - "z"), digits ("0" - "9"), underscores ("_"), and hyphens ("-"), and SHOULD NOT exceed 20 characters in length.

Type Description
: A brief description of the Subject Identifier Type.

Change Controller
: For types defined in documents published by the IETF or its working groups, list "IETF".  For all other types, list the name of the party responsible for the registration.  Contact information such as mailing address, email address, or phone number may also be provided.

Defining Document(s)
: A reference to the document or documents that define the Subject Identifier Type.  The definition MUST specify the name, format, and meaning of each member that may occur within a Subject Identifier of the defined type, as well as whether each member is optional, required, prohibited, or the circumstances under which the member may be optional, required, or prohibited. URIs that can be used to retrieve copies of each document SHOULD be included.

### Initial Registry Contents {#iana-sub-id-types-init}

#### Account Subject Identifier Type

* Type Name: `account`
* Type Description: Subject identifier based on `acct` URI.
* Change Controller: IETF
* Defining Document(s): {{sub-ids}} of this document.

#### Email Subject Identifier Type

* Type Name: `email`
* Type Description: Subject identifier based on email address.
* Change Controller: IETF
* Defining Document(s): {{sub-ids}} of this document.

#### Issuer and Subject Subject Identifier Type

* Type Name: `iss_sub`
* Type Description: Subject identifier based on an issuer and subject.
* Change Controller: IETF
* Defining Document(s): {{sub-ids}} of this document.

#### Phone Number Subject Identifier Type

* Type Name: `phone_number`
* Type Description: Subject identifier based on an phone number.
* Change Controller: IETF
* Defining Document(s): {{sub-ids}} of this document.

#### Aliases Subject Identifier Type

* Type Name: `aliases`
* Type Description: Subject identifier that groups together multiple different subject identifiers for the same subject.
* Change Controller: IETF
* Defining Document(s): {{sub-ids}} of this document.

### Guidance for Expert Reviewers {#iana-sub-id-types-expert}
The Expert Reviewer is expected to review the documentation referenced in a registration request to verify its completeness. The Expert Reviewer must base their decision to accept or reject the request on a fair and impartial assessment of the request. If the Expert Reviewer has a conflict of interest, such as being an author of a defining document referenced by the request, they must recuse themselves from the approval process for that request. In the case where a request is rejected, the Expert Reviewer should provide the requesting party with a written statement expressing the reason for rejection, and be prepared to cite any sources of information that went into that decision.

Subject Identifier Types need not be generally applicable and may be highly specific to a particular domain; it is expected that types may be registered for niche or industry-specific use cases. The Expert Reviewer should focus on whether the type is thoroughly documented, and whether its registration will promote or harm interoperability.  In most cases, the Expert Reviewer should not approve a request if the registration would contribute to confusion, or amount to a synonym for an existing type.
 
JSON Web Token Claims Registration
----------------------------------
This document defines the `sub_id` JWT Claim, which IANA is asked to register in the "JSON Web Token Claims" registry [IANA JSON Web Token Claims Registry](#IANA.JWT.Claims) established by {{!RFC7519}}.

### Registry Contents

* Claim Name: "sub_id"
* Claim Description: Subject Identifier
* Change Controller: IESG
* Specification Document(s): {{jwt-claims-sub_id}} of this document.

--- back

Acknowledgements
================
{: numbered="no"}
The authors would like to thank the members of the IETF Security Events working group, as well as those of the OpenID Shared Signals and Events Working Group, whose work provided the original basis for this document.

Change Log
==========
{: numbered="no"}
(This section to be removed by the RFC Editor before publication as an RFC.)

Draft 00 - AB - First draft

Draft 01 - AB:

* Added reference to RFC 5322 for format of `email` claim.
* Renamed `iss_sub` type to `iss-sub`.
* Renamed `id_token_claims` type to `id-token-claims`.
* Added text specifying the nature of the subjects described by each type.

Draft 02 - AB:

* Corrected format of phone numbers in examples.
* Updated author info.

Draft 03 - AB:

* Added `account` type for `acct` URIs.
* Replaced `id-token-claims` type with `aliases` type.
* Added email canonicalization guidance.
* Updated semantics for `email`, `phone`, and `iss-sub` types.

Draft 04 - AB:

* Added `sub_id` JWT Claim definition, guidance, examples.
* Added text prohibiting `aliases` nesting.
* Added privacy considerations for identifier correlation.

Draft 05 - AB:

* Renamed the `phone` type to `phone-number` and its `phone` claim to `phone_number`.

Draft 06 - AB:

* Replaced usage of the word "claim" to describe members of a Subject Identifier with the word "member", in accordance with terminology in RFC7159.
* Renamed the `phone-number` type to `phone_number` and `iss-sub` to `iss_sub`.
* Added normative requirements limiting the use of both `sub` and `sub_id` claims together when processing a JWT.
* Clarified that identifier correlation may be acceptable when it is a core part of the use case.
* Replaced references to OIDF with IETF in IANA Considerations.
* Recommended the appointment of multiple Designated Experts, and a location for the Subject Identifier Types registry.
* Added "_" to list of allowed characters in the Type Name for Subject Identifier Types.
* Clarified that Subject Identifiers don't provide confidentiality or integrity protection.
* Added references to SET, JWT privacy and security considerations.
* Added section describing the difference between subject identifier type and principal type that hopefully clarifies things and doesn't just muddy the water further.
