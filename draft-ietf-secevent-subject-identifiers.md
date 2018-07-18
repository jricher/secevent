---
title: Subject Identifiers for Security Event Tokens
abbrev: secevent-subject-identifiers
docname: draft-backman-secevent-subject-identifiers-00
date: 2018-06-01
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

 -
    ins: M. Scurtescu
    name: Marius Scurtescu
    organization: Google
    email: mscurtescu@google.com

normative:
  JSON: RFC7159
  JWT: RFC7519
  BCP26: RFC8126
  E164:
    title: The international public telecommunication numbering plan
    target: http://www.itu.int/rec/T-REC-E.164-201011-I/en
    author:
      org: International Telecommunication Union
    date: 2010
  IDTOKEN:
    title: OpenID Connect Core 1.0 - ID Token
    target: http://openid.net/specs/openid-connect-core-1_0.html#IDToken
    author:
     -
        ins: N. Sakamura
        name: Nat Sakamura
     -
        ins: J. Bradley
        name: John Bradley
     -
        ins: M. Jones
        name: Michael B. Jones
     -
        ins: B. de Medeiros
        name: Breno de Medeiros
     -
        ins: C. Mortimore
        name: Chuck Mortimore
    date: 2017-04-07
  SET:
    title: Security Event Token (SET)
    target: https://tools.ietf.org/html/draft-ietf-secevent-token-01

--- abstract

Security events communicated within Security Event Tokens may support a variety of identifiers to identify the subject and/or other principals related to the event.  This specification formalizes the notion of subject identifiers as named sets of well-defined claims describing the subject, a mechanism for representing subject identifiers within a {{!JSON}} object such as a [JSON Web Token (JWT)](#JWT) or [Security Event Token (SET)](#SET), and a registry for defining and allocating names for these claim sets.

--- middle

Introduction {#intro}
============
As described in section 1.2 of [SET](#SET), the subject of a security event may take a variety of forms, including but not limited to a JWT principal, an IP address, a URL, etc. Furthermore, even in the case where the subject of an event is more narrowly scoped, there may be multiple ways by which a given subject may be identified. For example, an account may be identified by an opaque identifier, an email address, a phone number, a JWT iss claim and sub claim, etc., depending on the nature and needs of the transmitter and receiver. Even within the context of a given transmitter and receiver relationship, it may be appropriate to identify different accounts in different ways, for example if some accounts only have email addresses assoicated with them while others only have phone numbers. Therefore it can be necessary to indicate within a SET the mechanism by which the subject of the security event is being identified.

Notational Conventions {#conv}
======================
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in {{!RFC2119}}.

Subject Identifiers {#sub-ids}
===================
A Subject Identifier Type is a light-weight schema that describes a set of claims that identifies a subject.  Every Subject Identifier Type MUST have a unique name registered in the IANA "Security Event Subject Identifier Types" registry established by {{iana-sub-id-types}}.  A Subject Identifier Type MAY describe more claims than are strictly necessary to uniquely identify a subject, and MAY describe conditions under which those claims are required, optional, or prohibited.

A Subject Identifier is a {{!JSON}} object containing a `subject_type` claim whose value is the unique name of a Subject Identifier Type, and a set of additional "payload claims" which are to be interpreted according to the rules defined by that Subject Identifier Type.  Payload claim values MUST match the format specified for the claim by the Subject Identifier Type. A Subject Identifier MUST NOT contain any payload claims prohibited or not described by its Subject Identifier Type, and MUST contain all payload claims required by its Subject Identifier Type.

The following Subject Identifier Types are registered in the IANA "Security Event Subject Identifier Types" registry established by {{iana-sub-id-types}}.

Email Subject Identifier Type {#sub-id-email}
-----------------------------
The Email Subject Identifier Type describes a subject by email address. Subject Identifiers of this type MUST contain an `email` claim whose value is a string containing the email address of the subject. The `email` claim MUST NOT be null or empty. The Email Subject Identifier Type is identified by the name `email`.

Below is a non-normative example Subject Identifier for the Email Subject Identifier Type:

~~~
{
  "subject_type": "email",
  "email": "user@example.com",
}
~~~
{: #figexamplesubidemail  title="Example: Subject Identifier for the Email Subject Identifier Type."}

Phone Number Subject Identifier Type {#sub-id-phone}
------------------------------------
The Phone Number Subject Identifier Type describes a subject by telephone number.  Subject Identifiers of this type MUST contain a `phone` claim whose value is a string containing the full telephone number of the subject, including international dialing prefix, formatted according to [E.164](#E164). The `phone` claim MUST NOT be null or empty. The Phone Number Subject Identifier Type is identified by the name `phone`.

Below is a non-normative example Subject Identifier for the Email Subject Identifier Type:

~~~
{
  "subject_type": "phone",
  "phone": "+1 (206) 555-0100",
}
~~~
{: #figexamplesubidphone  title="Example: Subject Identifier for the Phone Number Subject Identifier Type."}

Issuer and Subject Subject Identifier Type {#sub-id-iss-sub}
------------------------------------------
The Issuer and Subject Subject Identifier Type describes a subject by an issuer and a subject. Subject Identifiers of this type MUST contain an `iss` claim whose value identifies the issuer, and a `sub` claim whose value identifies the subject with respect to the issuer. These claims MUST follow the formats of the `iss` claim and `sub` claim defined by {{!RFC7519}}, respectively. Both the `iss` claim and the `sub` claim MUST NOT be null or empty. The Issuer and Subject Subject Identifier Type is identified by the name `iss_sub`.

Below is a non-normative example Subject Identifier for the Issuer and Subject Subject Identifier Type:

~~~
{
  "subject_type": "iss_sub",
  "iss": "http://issuer.example.com/",
  "sub": "145234573",
}
~~~
{: #figexamplesubidisssub  title="Example: Subject Identifier for the Issuer and Subject Subject Identifier Type."}

ID Token Claims Subject Identifier Type {#sub-id-id-token}
---------------------------------------
The ID Token Claims Subject Identifier Type describes a subject by a subset of the claims from an ID token. Subject Identifiers of this type MUST contain at least one of the following claims:

{: vspace="0"}
email
: An `email` claim, as defined in [IDTOKEN](#IDTOKEN).

phone_number
: A `phone_number` claim, as defined in [IDTOKEN](#IDTOKEN).

sub
: A `sub` claim, as defined in {{!RFC7519}}.

If the Subject Identifier contains a `sub` claim, it MUST also contain an `iss` claim, as defined in {{!RFC7519}}.  The ID Token Claims Subject Identifier Type is identified by the name `id_token_claims`.

Below is a non-normative example Subject Identifier for the ID Token Claims Subject Identifier Type:

~~~
{
  "subject_type": "id_token_claims",
  "iss": "http://issuer.example.com/",
  "sub": "145234573",
  "email": "user@example.com",
}
~~~
{: #figexamplesubididtoken  title="Example: Subject Identifier for the ID Token Claims Subject Identifier Type."}

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
: For types defined in documents published by the OpenID Foundation or its working groups, list "OIDF RISC Working Group".  For all other types, list the name of the party responsible for the registration.  Contact information such as mailing address, email address, or phone number may also be provided.

Defining Document(s)
: A reference to the document or documents that define the Subject Identifier Type.  The definition MUST specify the name, format, and meaning of each claim that may occur within a Subject Identifier of the defined type, as well as whether each claim is optional or required, or the circumstances under which the claim is optional or required. URIs that can be used to retrieve copies of each document SHOULD be included.

### Initial Registry Contents {#iana-sub-id-types-init}

#### Email Subject Identifier Type

* Type Name: `email`
* Type Description: Subject identifier based on email address.
* Change Controller: IETF secevent Working Group
* Defining Document(s): {{sub-ids}} of this document.

#### ID Token Claims Subject Identifier Type

* Type Name: `id_token_claims`
* Type Description: Subject identifier based on OpenID Connect ID Token claims.
* Change Controller: IETF secevent Working Group
* Defining Document(s): {{sub-ids}} of this document.

#### Issuer and Subject Subject Identifier Type

* Type Name: `iss_sub`
* Type Description: Subject identifier based on an issuer and subject.
* Change Controller: IETF secevent Working Group
* Defining Document(s): {{sub-ids}} of this document.

#### Phone Number Subject Identifier Type

* Type Name: `phone`
* Type Description: Subject identifier based on an phone number.
* Change Controller: IETF secevent Working Group
* Defining Document(s): {{sub-ids}} of this document.

### Guidance for Exper Reviewers {#iana-sub-id-types-expert }
The Expert Reviewer is expected to review the documentation referenced in a registration request to verify its completeness. The Expert Reviewer must base their decision to accept or reject the request on a fair and impartial assessment of the request. If the Expert Reviewer has a conflict of interest, such as being an author of a defining document referenced by the request, they must recuse themselves from the approval process for that request. In the case where a request is rejected, the Expert Reviewer should provide the requesting party with a written statement expressing the reason for rejection, and be prepared to cite any sources of information that went into that decision.

Subject Identifier Types need not be generally applicable and may be highly specific to a particular domain; it is expected that types may be registered for niche or industry-specific use cases. The Expert Reviewer should focus on whether the type is thoroughly documented, and whether its registration will promote or harm interoperability.  In most cases, the Expert Reviewer should not approve a request if the registration would contribute to confusion, or amount to a synonym for an existing type.

Privacy Considerations {#privacy}
======================
There are no privacy considerations.

Security Considerations {#security}
=======================
There are no security considerations.

--- back

Acknowledgements
================
{: numbered="no"}
This document is based on work developed within the OpenID RISC Working Group. The authors would like to thank the members of this group for their hard work and contributions.

Change Log
==========
{: numbered="no"}
(This sectcion to be removed by the RFC Editor before publication as an RFC.)

Draft 00 - AB - First draft
