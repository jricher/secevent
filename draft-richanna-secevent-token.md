---
title: Security Event Token (SET)
abbrev: secevent-token
docname: draft-richanna-secevent-token-00
date: 2017-11-14
category: info
ipr: trust200902

area: Security
workgroup: secevent
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
    ins: P. Hunt
    name: Phil Hunt
    organization: Oracle
    email: phil.hunt@yahoo.com
    
normative:
  IANA.JWT.Claims:
	title: JSON Web Token Claims
	target: http://www.iana.org/assignments/jwt
  IANA.MediaTypes:
	title: Media Types
	target: http://www.iana.org/assignments/media-types
  RFC2119:
  RFC3986:
  RFC5246:
  RFC6125:
  RFC6749:
  RFC7519:
  RFC7525:
  RFC7617:
informative:
  I-D.ietf-stir-passport:
  I-D.sheffer-oauth-jwt-bcp:
  OpenID.Core:
    title: OpenID Connect Core 1.0
    target: http://openid.net/specs/openid-connect-core-1_0.html

  RFC2046:
  RFC6838:
  RFC7009:
  RFC7515:
  RFC7516:
  RFC7517:
  RFC7644:
  RFC8055:
  saml-core-2.0:
    title: Assertions and Protocols for the OASIS Security Assertion Markup Language (SAML) V2.0

--- abstract
This specification defines the Security Event Token, which may be
distributed via a protocol such as HTTP. The Security Event Token
(SET) specification profiles the JSON Web Token (JWT), which can be
optionally signed and/or encrypted. A SET describes a statement of
fact from the perspective of an issuer that it intends to share with
one or more receivers.

--- middle

Introduction {#intro}
============
This specification defines an extensible Security Event Token (SET)
format which may be exchanged using protocols such as HTTP. The
specification builds on the JSON Web Token (JWT) format [RFC7519] in
order to provide a self-contained token that can be optionally signed
using JSON Web Signature (JWS) [RFC7515] and/or encrypted using JSON
Web Encryption (JWE) [RFC7516].

This specification profiles the use of JWT for the purpose of issuing
security event tokens (SETs). This specification defines a base
format upon which profiling specifications define actual events and
their meanings. Unless otherwise specified, this specification uses
non-normative example events intended to demonstrate how events may
be constructed.

This specification is scoped to security and identity related events.
While security event tokens may be used for other purposes, the
specification only considers security and privacy concerns relevant
to identity and personal information.

Security Events are not commands issued between parties. A security
event is a statement of fact from the perspective of an issuer about
the state of a security subject (e.g., a web resource, token, IP
address, the issuer itself) that the issuer controls or is aware of,
that has changed in some way (explicitly or implicitly). A security
subject MAY be permanent (e.g., a user account) or temporary (e.g.,
an HTTP session) in nature. A state change could describe a direct
change of entity state, an implicit change of state or other higher-
level security statements such as:

*  The creation, modification, removal of a resource.
*  The resetting or suspension of an account.
*  The revocation of a security token prior to its expiry.
*  The logout of a user session. Or,
*  A cumulative conclusion such as to indicate that a user has taken
   over an email identifier that may have been used in the past by
   another user.

While subject state changes are often triggered by a user-agent or
security-subsystem, the issuance and transmission of an event often
occurs asynchronously and in a back-channel to the action which
caused the change that generated the security event. Subsequently,
an Event Receiver, having received a SET, validates and interprets
the received SET and takes its own independent actions, if any. For
example, having been informed of a personal identifier being
associated with a different security subject (e.g., an email address
is being used by someone else), the Event Receiver may choose to
ensure that the new user is not granted access to resources
associated with the previous user. Or, the Event Receiver may not
have any relationship with the subject, and no action is taken.

While Event Receivers will often take actions upon receiving SETs,
security events cannot be assumed to be commands or requests. The
intent of this specification is to define a way of exchanging
statements of fact that Event Receivers may interpret for their own
purposes. As such, SETs have no capability for error signaling other
to ensure the validation of a received SET.

Notational Conventions {#conv}
----------------------
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
[RFC2119]. These keywords are capitalized when used to unambiguously
specify requirements of the protocol or application features and
behavior that affect the inter-operability and security of
implementations. When these words are not capitalized, they are
meant in their natural-language sense.

For purposes of readability, examples are not URL encoded.
Implementers MUST percent encode URLs as described in Section 2.1 of
[RFC3986].

Throughout this document, all figures MAY contain spaces and extra
line-wrapping for readability and space limitations. Similarly, some
URIs contained within examples have been shortened for space and
readability reasons.

Definitions {#defn}
-----------
The following definitions are used with SETs:

{: vspace="0"}
Security Event Token (SET)
:  A SET is a JWT [RFC7519] that is distributed to one or more
registered Event Receivers.

Event Transmitter
: A service provider that delivers SETs to other providers known as
Event Receivers.

Event Receiver
: An Event Receiver is an entity that receives SETs through some
distribution method. An Event Receiver is the same entity
referred as "recipient" or "receiver" in and related
specifications. [RFC7519]

Subject
: A SET describes an event or state change that has occurred about a
Subject. A Subject may be a principal (e.g., Section 4.1.2
[RFC7519]), a web resource, an entity such as an IP address, or
the issuer itself that a SET might reference.

Profiling Specification  A specification that uses the SET Token
specification to define one or more event types and the associated
claims included.

The Security Event Token (SET) {#set}
==============================
A SET is a data structure that consists of an "event payload" describing a
security event, wrapped in an "envelope" providing metadata and context
for the security event. The SET envelope is a JWT as defined in
[RFC7519], consisting of a JSON object containing a set of claims. The
event payload is a JSON object contained within the SET envelope, itself
containing claims that contain information about the event: the type
of event, the subject of the event, and other information defined in a
Profiling Specification.

This specification defines claims to be used in envelopes and claims to be
used in event payloads, however Profiling Specifications MAY define
additional claims for use in SETs. It is RECOMMENDED that Profiling
Specifications define claims to be used in the event payload rather than
the envelope. If a Profiling Specification does define envelope claims,
those claims SHOULD be registered in the JWT Token Claims Registry
[IANA.JWT.Claims] or have Public Claim Names as defined in Section 4.2 of
[RFC7519].

SET Claims {#claims}
----------
This specification profiles the following claims defined in [RFC7519] for
use in the SET envelope: 

{: vspace="0"}
iss
: A case-sensitive string identifying the principal that issued the SET,
as defined by Section 4.1.1 of [RFC7519]. This claim is REQUIRED.

aud
: A case-sensitive string or array of case-sensitive strings identifying
the audience for the SET, as defined by Section 4.1.3 of [RFC7519]. This
claim is RECOMMENDED.

exp
: As defined by Section 4.1.4 of [RFC7519], this claim is the time after
which the JWT MUST NOT be accepted for processing. In the context of a SET
however, this notion does not apply since a SET reflects something that
has already been processed and is historical in nature. Use of this claim
is NOT RECOMMENDED.

iat
: A value identifying the time at which the SET was issued, as defined by
Section 4.1.6 of [RFC7519]. This claim is REQUIRED.

jti
: A unique identifier for an event, as defined by Section 4.1.7 of
[RFC7519]. This claim is REQUIRED.

This specification defines the following new claims for use in SETs:

{: vspace="0"}
event
: A JSON object known as the "event payload", whose contents identify the
type of event contained within the SET and contain additional information 
defined as part of an event type definition in a Profiling Specification. 
This specification defines the following claims for use in event payloads:

  {: vspace="0"}
  event_type
  : A string containing a URI that uniquely identifies an event type
defined by a Profiling Specification. This claim is REQUIRED.

  sub
  : A JSON object whose contents identify the subject of the event. This
object SHOULD NOT contain more information than is necessary to enable the
receiver to identify the subject. This claim is RECOMMENDED.

  The event payload MAY contain additional claims, such as those defined
as part of the event type definition in a Profiling Specification. The
format and meaning of these claims is out of scope of this specification. 
Implementations SHOULD ignore any claims in the event payload that they do
not understand.

toe
: A number identifying the date and time at which the event is believed to
have occurred or will occur in the future. Its value MUST take the form of
a NumericDate value, as defined in Section 2 of [RFC7519]. This claim is
RECOMMENDED. Profiling Specifications MAY indicate ways to determine the
event time when the "toe" claim is omitted (such as using the value of the
"iat" claim).

txn
: A string value that represents a unique transaction identifier. In cases
where multiple SETs are issued containing different events, the
transaction identifier MAY be used to correlate the SETs to the same
originating event or stateful change. This claim is OPTIONAL.

The following is a non-normative example showing a SET containing a
hypothetical event with two additional claims in the payload.

~~~
{
  "jti": "3d0c3cf797584bd193bd0fb1bd4e7d30",
  "typ": "secevent+jwt",
  "iss": "https://transmitter.example.com",
  "aud": [ "https://receiver.example.com" ],
  "iat": 1458496025,
  "toe": 1458492425,
  "txn": "5bb4ddd2-3e77-4e0b-a406-03c8fdc287c2",
  "event": {
	"event_type": "https://secevent.example.com/example_event",
    "sub": {
      "email": "user@example.com"
    },
    "claim_1": "foo",
    "claim_2": "bar"
  }
}
~~~
{: #figset title="Example SET With Event Claims In Payload"}

The payload in this example contains the following:
* An "event_type" claim whose value is the URI identifying the
hypothetical event type.
* A "sub" claim whose value identifies a subject via email address.
* Two claims "claim_1" and "claim_2" that are defined by the hypothetical 
event type's Profiling Specification.

Explicit Typing of SETs {#set-type}
-----------------------
This specification registers the "application/secevent+jwt" media
type. SETs MAY include this media type in the "typ" header parameter of
the JWT containing the SET to explicitly declare that the JWT is a SET.
This MUST be included if the SET could be used in an application context
in which it could be confused with other kinds of JWTs.
Profiling Specifications MAY declare that this is REQUIRED for SETs
containing events defined by the Profiling Specification.

Per the definition of "typ" in Section 4.1.9 of [RFC7515], it is
RECOMMENDED that the "application/" prefix be omitted. Therefore,
the "typ" value used SHOULD be "secevent+jwt".

Security Event Token Construction {#construction}
---------------------------------
A SET is a JWT [RFC7519] that is constructed by building a JSON
structure that constitutes an event object which is then used as the
body of a JWT.

While this specification uses JWT to convey a SET, implementers SHALL
NOT use SETs to convey authentication or authorization assertions.

When transmitted, the above JSON body must be converted into a JWT as
per [RFC7519].

The following is an example of a SCIM Event expressed as an unsecured
JWT. The JOSE Header is:

~~~
{"typ":"secevent+jwt","alg":"none"}
~~~

Base64url encoding of the octets of the UTF-8 representation of the
JOSE Header yields:

~~~
eyJ0eXAiOiJzZWNldmVudCtqd3QiLCJhbGciOiJub25lIn0
~~~

The example JWT Claims Set is encoded as follows:

~~~
ewogICJqdGkiOiAiNGQzNTU5ZWM2NzUwNGFhYmE2NWQ0MGIwMzYzZmFhZDgiLAogICJp
YXQiOiAxNDU4NDk2NDA0LAogImlzcyI6ICJodHRwczovL3NjaW0uZXhhbXBsZS5jb20i
LAogICJhdWQiOiBbCiAgICAiaHR0cHM6Ly9zY2ltLmV4YW1wbGUuY29tL0ZlZWRzLzk4
ZDUyNDYxZmE1YmJjODc5NTkzYjc3NTQiLAogICAgImh0dHBzOi8vc2NpbS5leGFtcGxl
LmNvbS9GZWVkcy81ZDc2MDQ1MTZiMWQwODY0MWQ3Njc2ZWU3IgogIF0sCiAKICAiZXZl
bnQiOiB7CiAgICAiZXZlbnRfdHlwZSI6ICJ1cm46aWV0ZjpwYXJhbXM6c2NpbTpldmVu
dDpjcmVhdGUiLAogICAgInJlZiI6ICJodHRwczovL3NjaW0uZXhhbXBsZS5jb20vVXNl
cnMvNDRmNjE0MmRmOTZiZDZhYjYxZTc1MjFkOSIsCiAgICAiYXR0cmlidXRlcyI6IFsi
aWQiLCAibmFtZSIsICJ1c2VyTmFtZSIsICJwYXNzd29yZCIsICJlbWFpbHMiXQogIH0K
fQo=
~~~

The encoded JWS signature is the empty string. Concatenating the
parts yields:

~~~
eyJ0eXAiOiJzZWNldmVudCtqd3QiLCJhbGciOiJub25lIn0.
ewogICJqdGkiOiAiNGQzNTU5ZWM2NzUwNGFhYmE2NWQ0MGIwMzYzZmFhZDgiLAogICJp
YXQiOiAxNDU4NDk2NDA0LAogImlzcyI6ICJodHRwczovL3NjaW0uZXhhbXBsZS5jb20i
LAogICJhdWQiOiBbCiAgICAiaHR0cHM6Ly9zY2ltLmV4YW1wbGUuY29tL0ZlZWRzLzk4
ZDUyNDYxZmE1YmJjODc5NTkzYjc3NTQiLAogICAgImh0dHBzOi8vc2NpbS5leGFtcGxl
LmNvbS9GZWVkcy81ZDc2MDQ1MTZiMWQwODY0MWQ3Njc2ZWU3IgogIF0sCiAKICAiZXZl
bnQiOiB7CiAgICAiZXZlbnRfdHlwZSI6ICJ1cm46aWV0ZjpwYXJhbXM6c2NpbTpldmVu
dDpjcmVhdGUiLAogICAgInJlZiI6ICJodHRwczovL3NjaW0uZXhhbXBsZS5jb20vVXNl
cnMvNDRmNjE0MmRmOTZiZDZhYjYxZTc1MjFkOSIsCiAgICAiYXR0cmlidXRlcyI6IFsi
aWQiLCAibmFtZSIsICJ1c2VyTmFtZSIsICJwYXNzd29yZCIsICJlbWFpbHMiXQogIH0K
fQo=.
~~~
{: #figset title="Example Unsecured Security Event Token"}

For the purpose of a simpler example in Figure 5, an unsecured token
was shown. When SETs are not signed or encrypted, the Event Receiver
MUST employ other mechanisms such as TLS and HTTP to provide
integrity, confidentiality, and issuer validation, as needed by the
application.

When validation (i.e. auditing), or additional transmission security
is required, JWS signing and/or JWE encryption MAY be used. To
create and or validate a signed and/or encrypted SET, follow the
instructions in Section 7 of [RFC7519].

Requirements for SET Profiles {#profile-req}
=============================
Profiling Specifications for SETs define the syntax and semantics of
SETs conforming to that SET profile and rules for validating those
SETs. The syntax defined by profiling specifications includes what
claims and event payload values are used by SETs utilizing the
profile.

Defining the semantics of the SET contents for SETs utilizing the
profile is equally important. Possibly most important is defining
the procedures used to validate the SET issuer and to obtain the keys
controlled by the issuer that were used for cryptographic operations
used in the JWT representing the SET. For instance, some profiles
may define an algorithm for retrieving the SET issuer's keys that
uses the "iss" claim value as its input. Likewise, if the profile
allows (or requires) that the JWT be unsecured, the means by which
the integrity of the JWT is ensured MUST be specified.

Profiling Specifications MUST define how the event Subject is
identified in the SET, as well as how to differentiate between the
event Subject's Issuer and the SET Issuer, if applicable. It is NOT
RECOMMENDED for Profiling Specifications to use the "sub" claim in
cases in which the Subject is not globally unique and has a different
Issuer from the SET itself.

Profiling Specifications MUST clearly specify the steps that a
recipient of a SET utilizing that profile MUST perform to validate
that the SET is both syntactically and semantically valid.

Security Considerations {#security}
=======================

Confidentiality and Integrity {#c-and-i}
-----------------------------
SETs may often contain sensitive information. Therefore, methods for
distribution of events SHOULD require the use of a transport-layer
security mechanism when distributing events. Parties MUST support
TLS 1.2 [RFC5246] and MAY support additional transport-layer
mechanisms meeting its security requirements. When using TLS, the
client MUST perform a TLS/SSL server certificate check, per
[RFC6125]. Implementation security considerations for TLS can be
found in "Recommendations for Secure Use of TLS and DTLS" [RFC7525].

Security Events distributed through third-parties or that carry
personally identifiable information, SHOULD be encrypted using JWE
[RFC7516] or secured for confidentiality by other means.

Unless integrity of the JWT is ensured by other means, it MUST be
signed using JWS [RFC7515] so that individual events can be
authenticated and validated by the Event Receiver.

Delivery {#delivery}
--------
This specification does not define a delivery mechanism by itself.
In addition to confidentiality and integrity (discussed above),
implementers and Profiling Specifications MUST consider the
consequences of delivery mechanisms that are not secure and/or not
assured. For example, while a SET may be end-to-end secured using
JWE encrypted SETs, without TLS there is no assurance that the
correct endpoint received the SET and that it could be successfully
processed.

Sequencing {#sequencing}
----------
As defined in this specification, there is no defined way to order
multiple SETs in a sequence. Depending on the type and nature of SET
event, order may or may not matter. For example, in provisioning,
event order is critical -- an object could not be modified before it
was created. In other SET types, such as a token revocation, the
order of SETs for revoked tokens does not matter. If however, the
event was described as a log-in or logged-out status for a user
subject, then order becomes important.

Profiling Specifications and implementers SHOULD take caution when
using timestamps such as "iat" to define order. Distributed systems
will have some amount of clock-skew and thus time by itself will not
guarantee order.

Specifications profiling SET SHOULD define a mechanism for detecting
order or sequence of events. For example, the "txn" claim could
contain an ordered value (e.g., a counter) that the issuer defines.

Timing Issues {#timing}
-------------
When SETs are delivered asynchronously and/or out-of-band with
respect to the original action that incurred the security event, it
is important to consider that a SET might be delivered to an Event
Receiver in advance or well behind the process that caused the event.
For example, a user having been required to logout and then log back
in again, may cause a logout SET to be issued that may arrive at the
same time as the user-agent accesses a web site having just logged-
in. If timing is not handled properly, the effect would be to
erroneously treat the new user session as logged out. Profiling
Specifications SHOULD be careful to anticipate timing and subject
selection information. For example, it might be more appropriate to
cancel a "session" rather than a "user". Alternatively, the
specification could use timestamps that allows new sessions to be
started immediately after a stated logout event time.

Distinguishing SETs from ID Tokens {#not-id-tokens}
----------------------------------
Because [RFC7519] states that "all claims that are not understood by
implementations MUST be ignored", there is a consideration that a SET
token might be confused with ID Token [OpenID.Core] if a SET is
mistakenly or intentionally used in a context requiring an ID Token.
If a SET could otherwise be interpreted as a valid ID Token (because
it includes the required claims for an ID Token and valid issuer and
audience claim values for an ID Token) then that SET profile MUST
require that the "exp" claim not be present in the SET. Because
"exp" is a required claim in ID Tokens, valid ID Token
implementations will reject such a SET if presented as if it were an
ID Token.

Excluding "exp" from SETs that could otherwise be confused with ID
Tokens is actually defense in depth. In any OpenID Connect contexts
in which an attacker could attempt to substitute a SET for an ID
Token, the SET would actually already be rejected as an ID Token
because it would not contain the correct "nonce" claim value for the
ID Token to be accepted in contexts for which substitution is
possible.

Note that the use of explicit typing, as described in Section 2.2,
will not achieve disambiguation between ID Tokens and SETs, as the ID
Token validation rules do not use the "typ" header parameter value.

Distinguishing SETs from Access Tokens {#not-access-tokens}
--------------------------------------
OAuth 2.0 [RFC6749] defines access tokens as being opaque.
Nonetheless, some implementations implement access tokens as JWTs.
Because the structure of these JWTs is implementation-specific,
ensuring that a SET cannot be confused with such an access token is
therefore likewise, in general, implementation specific.
Nonetheless, it is recommended that SET profiles employ the following
strategies to prevent possible substitutions of SETs for access
tokens in contexts in which that might be possible:

*  Prohibit use of the "exp" claim, as is done to prevent ID Token
   confusion.

*  Where possible, use a separate "aud" claim value to distinguish
   between the Event Receiver and the protected resource that is the
   audience of an access token.

*  Modify access token validation systems to check for the presence
   of the "events" claim as a means to detect security event tokens.
   This is particularly useful if the same endpoint may receive both
   types of tokens.

*  Employ explicit typing, as described in Section 2.2, and modify
   access token validation systems to use the "typ" header parameter
   value.

Distinguishing SETs from other kinds of JWTs {#not-other-jwts}
--------------------------------------------
JWTs are now being used in application areas beyond the identity
applications in which they first appeared. For instance, the Session
Initiation Protocol (SIP) Via Header Field [RFC8055] and Personal
Assertion Token (PASSporT) [I-D.ietf-stir-passport] specifications
both define JWT profiles that use mostly or completely different sets
of claims than are used by ID Tokens. If it would otherwise be
possible for an attacker to substitute a SET for one of these (or
other) kinds of JWTs, then the SET profile must be defined in such a
way that any substituted SET will result in its rejection when
validated as the intended kind of JWT.

The most direct way to prevent confusion is to employ explicit
typing, as described in Section 2.2, and modify applicable token
validation systems to use the "typ" header parameter value. This
approach can be employed for new systems but may not be applicable to
existing systems.

Another way to ensure that a SET is not confused with another kind of
JWT is to have the JWT validation logic reject JWTs containing an
"events" claim unless the JWT is intended to be a SET. This approach
can be employed for new systems but may not be applicable to existing
systems.

For many use cases, the simplest way to prevent substitution is
requiring that the SET not include claims that are required for the
kind of JWT that might be the target of an attack. For example, for
[RFC8055], the "sip_callid" claim could be omitted and for
[I-D.ietf-stir-passport], the "orig" claim could be omitted.

In many contexts, simple measures such as these will accomplish the
task, should confusion otherwise even be possible. Note that this
topic is being explored in a more general fashion in JSON Web Token
Best Current Practices [I-D.sheffer-oauth-jwt-bcp]. The proposed
best practices in that draft may also be applicable for particular
SET profiles and use cases.

Privacy Considerations {#privacy}
======================
If a SET needs to be retained for audit purposes, JWS MAY be used to
provide verification of its authenticity.

Event Transmitters SHOULD attempt to specialize feeds so that the
content is targeted to the specific business and protocol needs of an
Event Receiver.

When sharing personally identifiable information or information that
is otherwise considered confidential to affected users, Event
Transmitters and Receivers MUST have the appropriate legal agreements
and user consent or terms of service in place.

The propagation of subject identifiers can be perceived as personally
identifiable information. Where possible, Event Transmitters and
Receivers SHOULD devise approaches that prevent propagation -- for
example, the passing of a hash value that requires the Event Receiver
to know the subject.

IANA Considerations {#iana}
===================

JSON Web Token Claims Registration {#iana-claims}
----------------------------------
This specification registers the "events", "toe", and "txn" claims in
the IANA "JSON Web Token Claims" registry [IANA.JWT.Claims]
established by [RFC7519].

### Registry Contents
*  Claim Name: "events"
*  Claim Description: Security Event URI
*  Change Controller: IESG
*  Specification Document(s): Section 2.1 of [[ this specification ]]

*  Claim Name: "toe"
*  Claim Description: Time Of Event
*  Change Controller: IESG
*  Specification Document(s): Section 2.1 of [[ this specification ]]

*  Claim Name: "txn"
*  Claim Description: Transaction Identifier
*  Change Controller: IESG
*  Specification Document(s): Section 2.1 of [[ this specification ]]

Media Type Registration {#iana-media-type}
-----------------------

### Registry Contents
This section registers the "application/secevent+jwt" media type
[RFC2046] in the "Media Types" registry [IANA.MediaTypes] in the
manner described in [RFC6838], which can be used to indicate that the
content is a SET.

*  Type name: application
*  Subtype name: secevent+jwt
*  Required parameters: n/a
*  Optional parameters: n/a
*  Encoding considerations: 8bit; A SET is a JWT; JWT values are
   encoded as a series of base64url-encoded values (some of which may
   be the empty string) separated by period ('.') characters.
*  Security considerations: See the Security Considerations section
   of [[ this specification ]]
*  Interoperability considerations: n/a
*  Published specification: Section 2.2 of [[ this specification ]]
*  Applications that use this media type: TBD
*  Fragment identifier considerations: n/a
*  Additional information:
  * Magic number(s): n/a
  * File extension(s): n/a
  * Macintosh file type code(s): n/a

*  Person & email address to contact for further information:
   Michael B. Jones, mbj@microsoft.com
*  Intended usage: COMMON
*  Restrictions on usage: none
*  Author: Michael B. Jones, mbj@microsoft.com
*  Change controller: IESG
*  Provisional registration?  No


--- back
Acknowledgments {#ack}
===============
The editors would like to thank the participants in the IETF secevent
mailing list and related working groups for their support of this
specification.

