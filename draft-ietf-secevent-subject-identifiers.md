---
title: Subject Identifiers for Security Event Tokens
abbrev: secevent-subject-identifiers
docname: draft-ietf-secevent-subject-identifiers-02
date: 2019-03-11
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
  JSON: RFC7159
  JWT: RFC7519
  SET: RFC8417
  E164:
    title: The international public telecommunication numbering plan
    target: http://www.itu.int/rec/T-REC-E.164-201011-I/en
    author:
      org: International Telecommunication Union
    date: 2010

--- abstract

Security events communicated within Security Event Tokens may support a variety of identifiers to identify the subject and/or other principals related to the event.  This specification formalizes the notion of subject identifiers as named sets of well-defined claims describing the subject, a mechanism for representing subject identifiers within a {{!JSON}} object such as a [JSON Web Token](#JWT) or [Security Event Token](#SET), and a registry for defining and allocating names for these claim sets.

--- middle

Introduction {#intro}
============
As described in section 1.2 of {{!SET}}, the subject of a security event may take a variety of forms, including but not limited to a JWT principal, an IP address, a URL, etc. Furthermore, even in the case where the subject of an event is more narrowly scoped, there may be multiple ways by which a given subject may be identified. For example, an account may be identified by an opaque identifier, an email address, a phone number, a JWT `iss` claim and `sub` claim, etc., depending on the nature and needs of the transmitter and receiver. Even within the context of a given transmitter and receiver relationship, it may be appropriate to identify different accounts in different ways, for example if some accounts only have email addresses associated with them while others only have phone numbers. Therefore it can be necessary to indicate within a SET the mechanism by which the subject of the security event is being identified.

Notational Conventions {#conv}
======================
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in {{!RFC2119}}.

Subject Identifiers {#sub-ids}
===================
A Subject Identifier Type is a light-weight schema that describes a set of claims that identifies a subject.  Every Subject Identifier Type MUST have a unique name registered in the IANA "Security Event Subject Identifier Types" registry established by {{iana-sub-id-types}}.  A Subject Identifier Type MAY describe more claims than are strictly necessary to identify a subject, and MAY describe conditions under which those claims are required, optional, or prohibited.

A Subject Identifier is a {{!JSON}} object containing a `subject_type` claim whose value is the name of a Subject Identifier Type, and a set of additional "payload claims" which are to be interpreted according to the rules defined by that Subject Identifier Type.  Payload claim values MUST match the format specified for the claim by the Subject Identifier Type. A Subject Identifier MUST NOT contain any payload claims prohibited or not described by its Subject Identifier Type, and MUST contain all payload claims required by its Subject Identifier Type.

The following Subject Identifier Types are registered in the IANA "Security Event Subject Identifier Types" registry established by {{iana-sub-id-types}}.

Account Subject Identifier Type {#sub-id-acct}
-------------------------------
The Account Subject Identifier Type describes a user account at a service provider, identified with an `acct` URI as defined in {{!RFC7565}}.  Subject Identifiers of this type MUST contain a `uri` claim whose value is the `acct` URI for the subject.  The `uri` claim is REQUIRED and MUST NOT be null or empty.  The Account Subject Identifier Type is identified by the name `account`.

Below is a non-normative example Subject Identifier for the Account Subject Identifier Type:

~~~
{
  "subject_type": "account",
  "uri": "acct:example.user@service.example.com",
}
~~~
{: #figexamplesubidaccount title="Example: Subject Identifier for the Account Subject Identifier Type."}

Email Subject Identifier Type {#sub-id-email}
-----------------------------
The Email Subject Identifier Type describes a principal identified with an email address.  Subject Identifiers of this type MUST contain an `email` claim whose value is a string containing the email address of the subject, formatted as an `addr-spec` as defined in Section 3.4.1 of {{!RFC5322}}. The `email` claim is REQUIRED and MUST NOT be null or empty. The value of the `email` claim SHOULD identify a mailbox to which email may be delivered, in accordance with {{!RFC5321}}. The Email Subject Identifier Type is identified by the name `email`.

Below is a non-normative example Subject Identifier for the Email Subject Identifier Type:

~~~
{
  "subject_type": "email",
  "email": "user@example.com",
}
~~~
{: #figexamplesubidemail  title="Example: Subject Identifier for the Email Subject Identifier Type."}

### Email Canonicalization {#email-canon}
Many email providers will treat multiple email addresses as equivalent. For example, some providers treat email addresses as case-insensitive, and consider "user@example.com", "User@example.com", and "USER@example.com" as the same email address. This has led users to view these strings as equivalent, driving service providers to implement proprietary email canonicalization algorithms to ensure that email addresses entered by users resolve to the same canonical string. When receiving an Email Subject Identifier, the recipient SHOULD use their implementation's canonicalization algorithm to resolve the email address to the same subject identifier string used in their system.

Phone Number Subject Identifier Type {#sub-id-phone}
------------------------------------
The Phone Number Subject Identifier Type describes a principal identified with a telephone number.  Subject Identifiers of this type MUST contain a `phone` claim whose value is a string containing the full telephone number of the subject, including international dialing prefix, formatted according to [E.164](#E164). The `phone` claim is REQUIRED and MUST NOT be null or empty. The Phone Number Subject Identifier Type is identified by the name `phone`.

Below is a non-normative example Subject Identifier for the Email Subject Identifier Type:

~~~
{
  "subject_type": "phone",
  "phone": "+12065550100",
}
~~~
{: #figexamplesubidphone  title="Example: Subject Identifier for the Phone Number Subject Identifier Type."}

Issuer and Subject Subject Identifier Type {#sub-id-iss-sub}
------------------------------------------
The Issuer and Subject Subject Identifier Type describes a principal identified with a pair of `iss` and `sub` claims, as defined by {{!JWT}}.  These claims MUST follow the formats of the `iss` claim and `sub` claim defined by {{!JWT}}, respectively. Both the `iss` claim and the `sub` claim are REQUIRED and MUST NOT be null or empty. The Issuer and Subject Subject Identifier Type is identified by the name `iss-sub`.

Below is a non-normative example Subject Identifier for the Issuer and Subject Subject Identifier Type:

~~~
{
  "subject_type": "iss-sub",
  "iss": "http://issuer.example.com/",
  "sub": "145234573",
}
~~~
{: #figexamplesubidisssub  title="Example: Subject Identifier for the Issuer and Subject Subject Identifier Type."}

Aliases Subject Identifier Type {#sub-id-aliases}
---------------------------------------
The Aliases Subject Identifier Type describes a subject that is identified with a list of different Subject Identifiers. It is intended for use when a variety of identifiers have been shared with the party that will be interpreting the Subject Identifier, and it is unknown which of those identifiers they will recognize or support.  Subject Identifiers of this type MUST contain an `identifiers` claim whose value is a JSON array containing one or more Subject Identifiers.  Each Subject Identifier in the array MUST identify the same entity.  The `identifiers` claim is REQUIRED and MUST NOT be null or empty.  It MAY contain multiple instances of the same Subject Identifier Type (e.g., multiple Email Subject Identifiers), but SHOULD NOT contain exact duplicates.  This type is identified by the name `aliases`.

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
      "subject_type": "phone",
      "phone": "+12065550100",
    },
    {
      "subject_type": "email",
      "email": "user+qualifier@example.com",
    }
  ],
}
~~~
{: #figexamplesubididtoken  title="Example: Subject Identifier for the Aliases Subject Identifier Type."}

Privacy Considerations {#privacy}
======================
There are no privacy considerations.

Security Considerations {#security}
=======================
There are no security considerations.

IANA Considerations {#iana}
===================

Security Event Subject Identifier Types Registry {#iana-sub-id-types}
------------------------------------------------
This document defines Subject Identifier Types, for which IANA is asked to create and maintain a new registry titled "Security Event Subject Identifier Types". Initial values for the Security Event Subject Identifier Types registry are given in {{sub-ids}}.  Future assignments are to be made through the Expert Review registration policy {{!BCP26}} and shall follow the template presented in {{iana-sub-id-types-template}}.

### Registration Template {#iana-sub-id-types-template}

{: vspace="0"}
Type Name
: The name of the Subject Identifier Type, as described in {{sub-ids}}. The name MUST be an ASCII string consisting only of lower-case characters ("a" - "z"), digits ("0" - "9"), and hyphens ("-"), and SHOULD NOT exceed 20 characters in length.

Type Description
: A brief description of the Subject Identifier Type.

Change Controller
: For types defined in documents published by the OpenID Foundation or its working groups, list "OpenID Foundation RISC Working Group".  For all other types, list the name of the party responsible for the registration.  Contact information such as mailing address, email address, or phone number may also be provided.

Defining Document(s)
: A reference to the document or documents that define the Subject Identifier Type.  The definition MUST specify the name, format, and meaning of each claim that may occur within a Subject Identifier of the defined type, as well as whether each claim is optional or required, or the circumstances under which the claim is optional or required. URIs that can be used to retrieve copies of each document SHOULD be included.

### Initial Registry Contents {#iana-sub-id-types-init}

#### Account Subject Identifier Type

* Type Name: `account`
* Type Description: Subject identifier based on `acct` URI.
* Change Controller: IETF secevent Working Group
* Defining Document(s): {{sub-ids}} of this document.

#### Email Subject Identifier Type

* Type Name: `email`
* Type Description: Subject identifier based on email address.
* Change Controller: IETF secevent Working Group
* Defining Document(s): {{sub-ids}} of this document.

#### Issuer and Subject Subject Identifier Type

* Type Name: `iss-sub`
* Type Description: Subject identifier based on an issuer and subject.
* Change Controller: IETF secevent Working Group
* Defining Document(s): {{sub-ids}} of this document.

#### Phone Number Subject Identifier Type

* Type Name: `phone`
* Type Description: Subject identifier based on an phone number.
* Change Controller: IETF secevent Working Group
* Defining Document(s): {{sub-ids}} of this document.

#### Aliases Subject Identifier Type

* Type Name: `aliases`
* Type Description: Subject identifier that groups together multiple different subject identifiers for the same subject.
* Change Controller: IETF secevent Working Group
* Defining Document(s): {{sub-ids}} of this document.

### Guidance for Expert Reviewers {#iana-sub-id-types-expert}
The Expert Reviewer is expected to review the documentation referenced in a registration request to verify its completeness. The Expert Reviewer must base their decision to accept or reject the request on a fair and impartial assessment of the request. If the Expert Reviewer has a conflict of interest, such as being an author of a defining document referenced by the request, they must recuse themselves from the approval process for that request. In the case where a request is rejected, the Expert Reviewer should provide the requesting party with a written statement expressing the reason for rejection, and be prepared to cite any sources of information that went into that decision.

Subject Identifier Types need not be generally applicable and may be highly specific to a particular domain; it is expected that types may be registered for niche or industry-specific use cases. The Expert Reviewer should focus on whether the type is thoroughly documented, and whether its registration will promote or harm interoperability.  In most cases, the Expert Reviewer should not approve a request if the registration would contribute to confusion, or amount to a synonym for an existing type.

--- back

Acknowledgements
================
{: numbered="no"}
This document is based on work developed within the OpenID RISC Working Group. The authors would like to thank the members of this group for their hard work and contributions.

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
