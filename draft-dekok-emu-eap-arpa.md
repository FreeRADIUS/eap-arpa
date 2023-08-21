---
title: The eap.arpa domain and EAP provisioning
abbrev: eap.arpa
docname: draft-dekok-emu-eap-arpa-00
updates: 9140

stand_alone: true
ipr: trust200902
area: Internet
wg: EMY Working Group
kw: Internet-Draft
cat: std
submissionType: IETF

pi:    # can use array (if all yes) or hash here
  toc: yes
  sortrefs:   # defaults to yes
  symrefs: yes

author:

- ins: A. DeKok
  name: Alan DeKok
  org: FreeRADIUS
  email: aland@freeradius.org

normative:
  BCP14: RFC8174
  RFC3748:
  RFC5216:
  RFC7542:
  RFC8126:
  RFC9140:
  RFC9190:

informative:
  RFC2865:
  RFC7170:
  RFC8952:
  RFC9191:
  I-D.friel-tls-eap-dpp:

venue:
  group: EMU
  mail: emut@ietf.org
  github: freeradius/eap-arpa.git

--- abstract

This document defines the eap.arpa domain as a way for EAP peers to
signal to EAP servers that they wish to obtain limited, and
unauthenticated, network access.  EAP peers leverage user identifier
portion of the Network Access Identifier (NAI) format of RFC7542 in
order to describe what kind of provisioning they need.  A table of
identifiers and meanings is defined.

--- middle

# Introduction

In most uses, EAP {{RFC3748}} requires that the EAP peer have a known
identity.  However, when the peer does not already have an identity,
this requirement creates a bootstrapping problem.  It may not be
possible for the device to obtain network access without credentials.
However, credentials are usually required in order to obtain network
access.  As a result, the device is unprovisioned, and unable to be
provisioned.

This specification addresses that problem.  It creates a framework by
which clients can submit predefined credentials to a server in order to
obtain limited network access.  At the same time, servers can know in
advance that these credentials are only to be used for provisioning,
and that unrestricted network access should not be granted.

The device can either use the EAP channel itself for provisioning, as
with TEAP {{RFC7170}}, or the EAP server can give the device access to
a limited captive portal such as with {{RFC8952}}.  Once the device is
provisioned, it can use those provisioned credentials to obtain full
network access.

The identifiers used here are generically referred to as "EAP
Provisioning Identifiers" (EPI).  The choice of "Provisioning
Identifiers for EAP" (PIE) was considered and rejected.

# Terminology

{::boilerplate bcp14}

# Concepts

A device which has no device-specific credentials can use a predefined
identifier in Network Access Identifier (NAI) format {{RFC7542}}.  The
NAI is composed of two portions, the utf8-username, and the utf8-realm
domain.  For simplicity here, we refer to these as the "username" and
"realm" fields.

The realm is chosen to be non-routable, so that the EAP packet
exchange is not sent across an Authentication, Authorization, and
Accounting (AAA) proxy framework as defined in {{RFC7542}} Section 3.
Instead, the packets remains local to the EAP server.  If the EAP
server implements this standard, then it can proceed with the full EAP
conversation.  If the EAP server does not implement this standard,
then it MUST reply with an EAP Failure, as per {{RFC3748}} Section
4.2.

We note that this specification is fully compatible with all existing
EAP implementations, so its is fail-safe.  When presented with a peer
wishing to use this specification, existing implementations will
return EAP Failure, and will not otherwise misbehave.

We now discuss the NAI format in more detail.  We first discuss the
eap.arpa realm, and second the use and purpose of the username field.

## The eap.arpa realm

This document defines the "eap.arpa" domain as being used for
provisioning within EAP.  A similar domain has previously been used
for EAP-NOOB {{RFC9140}}, as "eap-noob.arpa".  This document extends
that concept, and standardizes the practices surrounding it,

NOTE: the "arpa" domain is controlled by the IAB.  Allocation of
"eap.arpa" requires agreement from the IAB.

## The username field

The username field is assigned via the EAP Provisioning Identifier
Registry which is defined below.  The registry can either hold a fixed
string such as "noob", or a prefix such as "V-".  Prefixes give
vendors and Service delivery organizations (SDOs) the ability to
self-assign a delegated range of identifiers which cannot conflict
with other identifiers.

The username field MUST NOT omitted.  That is, "@eap.arpa" is not a
valid identifier for the purposes of this specification.  {{RFC7542}}
recommends omitting the username portion for user privacy.  As user
privacy is not needed here, the username field can be publicly visible.

# Overview

For EAP-TLS, both {{RFC5216}} Section 2.1.1 and {{RFC9190}} provide
for "peer unauthenticated access".  However, those documents define no
way for a peer to signal that it is requesting such access.  The
presumption is that the peer connects with some value for the EAP
Identity, but without using a client certificate.  The EAP server is
then supposed to determine that the peer is requesting unauthenticated
access, and take the approprate steps to limit authorization.

There appears to be no EAP peer or server implementations which
support such access, since there is no defined way to perform any of
the steps required.  i.e. to signal that this access is desired, and
then limit access.

TBD: The Wireless Broadband Alliance (WBA) has defined an unauthenticated
EAP-TLS method, using a vendor-specific EAP type. Get link.

EAP-NOOB {{RFC9140}} takes this process a step further.  It defines both
a way to signal that provisioning is desired, and also a way to
exchange provisioning information within EAP-NOOB.  That is, there is
no need for the device to obtain limited network access, as all of the
provisioning is done inside of the EAP-NOOB protocol.

TEAP {{RFC7170}} provides for provisioning via an unauthenticated TLS
tunnel.  There is a server unauthenticated provisioning mode (TBD),
but the inner TLS exchange requires that both end authenticate each
other.  There are ways to provision a certificate, but the peer must
still authenticate itself to the server.

## Taxonomy of Provisioning Types

There are two scenarios where provisioning can be done.  The first is
where provisioning is done within the EAP type, as with EAP-NOOB
{{RFC9140}}.  The second is where EAP is used to obtain limited
network access (e.g. as with a captive portal).  That limited network
access is then used to run Internet Protocol (IP) based provisioning
over more complex protocols.

### Rationale for Provisioning over EAP

It is often useful to do all provisioning inside of EAP, because the EAP / AAA
admin does not have control over the network.  It is not always
possible to define a captive portal where provisioning can be done.
As a result, we need to be able to perform provisioning via EAP, and
not via some IP protocol.

# Interaction with existing EAP types

As the provisioning identifer is used within EAP, it necessarily has
interactions with, and effects on, the various EAP types.  This
section discusses those effects in more detail.

Some EAP methods require shared credentials such as passwords in order
to succeed.  For example, both EAP-MSCHAPv2 (PEAP) and EAP-PWD
{{?RFC5931}} perform cryptographic exchanges where both parties
knowing a shared password.  Where those methods are used, the password
MUST be the same as the provisioning identifier.

This requirement also applies to TLS-based EAP methods such as TTLS
and PEAP.  Where the TLS-based EAP method provides for an inner
identity and inner authentication method, the credentials used there
MUST be the provisioning identifier for both the inner identity, and
any inner password.

This process ensures that most EAP methods will work for provisioning,
at the expense of potential security attacks.  TBD - discuss.

It is RECOMMENDED that provisioning be done via a TLS-based EAP methods.
TLS provides for authentication of the EAP server, along with security
and confidentiality of any provisioning data exchanged in the tunnel.
Similarly if provisioning is done in a captive portal outside of EAP,
EAP-TLS permits the EAP peer to run a full EAP authentication session
while having nothing more than a few certification authorities (CAs)
locally configured.

## EAP-TLS

This document defines an identifier "portal@eap.arpa", which is the
first step towards permitted unauthenticated client provisioning in
EAP-TLS.  The purpose of the identifier is to allow EAP peers to
signal EAP servers that they wish to obtain a "captive portal" style
network access.

This identifier signals the EAP server that the peer wishes to obtain
"peer unauthenticated access" as per {{RFC5216}} Section 2.1.1 and
{{RFC9190}}.

An EAP server which agrees to authenticate this request MUST ensure
that the device is placed into a captive portal with limited network
access.  Further details of the captive portal architecture can be
found in {{RFC8952}}.

The remaining question is how the EAP peer verifies the identity of
the EAP server.  The device SHOULD ignore the EAP server certificate
entirely, as the servers identity does not matter.  Any verification
of servers can be done at the HTTPS layer when the device access the
captive portal.  Where possible the device SHOULD warn the end user
that the provided certificate is unchecked, and untrusted.

However, since the device likely is configured with web CAs (otherwise
the captive portal would also be unauthenticated), EAP peers MAY use
the CAs available for the web in order to validate the EAP server
certificate.  If the presented certificate passes validation, the
device does not need to warn the end user that the provided
certificate is untrusted.

## TLS-based EAP methods

Other TLS-based EAP methods such as TTLS and PEAP can use the same
method as defined for EAP-TLS above.  The only difference is that the
inner identity and password is also the provisioning identifier.

Is is RECOMMENDED that provisioning methods use EAP-TLS in preference
to any other TLS-based EAP methods.  As the credentials for other
methods are predefined and known in advanc, those methods offer little
benefit over EAP-TLS.

## EAP-NOOB

It is RECOMMENDED that server implementations of EAP-NOOB accept both
identities "noob@eap-noob.arpa" and "noob@eap.arpa" as synonyms.

It is RECOMMENDED that EAP-NOOB peers use "noob@eap.arpa" first, and
if that does not succeed, use "noob@eap-noob.arpa"

@todo - what is the deployment of EAP-NOOB?  Can we even make this
recommendation?

# IANA Considerations

Three IANA actions are required.  The first two are registry updates
for "eap.arpa".  The second is the creation of a new registry.

## .arpa updates

IANA is instructed to update the ".ARPA Zone Management" registry with
the following entry:

DOMAIN

> eap.arpa

USAGE

> For provisioning within the Extensible Authentication Protocol framework. 

REFERENCE

> THIS-DOCUMENT

IANA is instructed to update the "Special-Use Domain Names" registry as follows:

NAME

> eap.arpa

REFERENCE

> THIS-DOCUMENT

## EAP Provisioning Identifier Registry

IANA is instructed to update the "Extensible Authentication Protocol
(EAP) Registry" with a new child registry.

Assignments in this registry are done via "Expert Review" as described in {{RFC8126}} Section 4.5.

The contents of the registry are as follows.

Title

> EAP Provisioning Identifier Registry

Registration Procedure(s)

> Expert review

Reference

> THIS-DOCUMENT

Registry

> Name
>
>> The name of the identifier.  Names are listed in sorted order, case insensitive.
>
> Prefix
>
>> A boolean true/false flag indicating if this name is a prefix.
>
> Description
>
>> Description of the use-cases for this identifier.
>
> Reference
>
>> Reference where this identifier was defined.

### Initial Values

The following table gives the initial values for this table.

Value,Prefix,Description,Reference

noob,false,EAP-NOOB,RFC9140 and THIS-DOCUMENT
portal,false,generic captive portal,THIS-DOCUMENT
V-,true,reserved for vendor self-assignment,THIS-DOCUMENT

## Guidelines for Designated Experts

Identifiers and prefixes in the "Name" field of this registry MUST
satisfy the "utf8-username" format defined in {{RFC7542}} Section 2.2.

Identifiers should be unique when compared in a case-insensitive
fashion.  While {{RFC7542}} notes that similar identifiers of
different case can be considered to be different, this registry is
made simpler by requiring case-insensitivity.

Identifiers and prefixes should be short.  The NAIs created from these
prefixes will generally be sent in a RADIUS packet in the User-Name
attribute ({{RFC2865}} Section 5.1).  That specification recommends
that implementations should support User-Names of at least 63 octets.
NAI length considerations are further discussed in {{RFC7542}} Section
2.3, and any allocations need to take those limitations into
consideration.

Implementations are likely to support a total NAI length of 63 octets.
Lengths between 63 and 253 octets may work.  Lengths of 254 octets or
more will not work with RADIUS {{RFC2865}}.

For registration requests where a Designated Expert should be
consulted, the responsible IESG area director should appoint the
Designated Expert.  The intention is that any non-prefix allocation
will be accompanied by a published RFC.  But in order to allow for the
allocation of values prior to the RFC being approved for publication,
the Designated Expert can approve allocations once it seems clear that
an RFC will be published.

For allocation of a prefix, the Designated Expert should verify that
the requested prefix clearly identifies the organization requesting
the prefix, that there is a publicly available document from the
organization which describes the prefix, and that the prefix ends with
the "-" character.

Once a prefix has been assigned, it is not possible to perform further
allocations in this registry which use that prefix.  All such
allocations have instead been delegated to the external organization.

The Designated expert will post a request to the EMU WG mailing list
(or a successor designated by the Area Director) for comment and
review, including an Internet-Draft or reference to external
specification.  Before a period of 30 days has passed, the Designated
Expert will either approve or deny the registration request and
publish a notice of the decision to the EAP WG mailing list or its
successor, as well as informing IANA.  A denial notice must be
justified by an explanation, and in the cases where it is possible,
concrete suggestions on how the request can be modified so as to
become acceptable should be provided.

A short-hand summary of the requirements follows:

* Identifiers and prefixes in the "Name" field of this registry MUST satisfy the "utf8-username" format defined in {{RFC7542}} Section 2.2.

* Names MUST be unique, compared in a case-insensitive fashion.

* Prefixes MUST NOT overlap with the beginning any other identifier.  That is, if the prefix "foo-" has been allocated, then an identifier of "foo-bar" MUST NOT be allocated.

* If the "prefix" flag is "false", then the Name field MUST NOT end with the "-" character.

* If the "prefix" flag is "true", then the Name field MUST end with the "-" character.

### Example of Vendor Self Assignment

Identifiers beginning with "V-" are for vendor self-assignment.  The
name MUST begin with the string "V-", following by 1 or more digits
(0-9).  The digits used here are taken from the vendor private
enterprise number (PEN).

The name MUST then contain another "-" which delineates the vendor
specific suffix namespace.  The suffix is managed and allocated by the
vendor, and does not need to be added to the registry.

The suffix is text which matches the "dot-string" definition of
{{RFC7542}} Section 2.2.

For example, an identifier chosen by Cisco (PEN of 9) could be:

> V-9-foo

Which then creates an NAI of the form:

> V-9-foo@eap.arpa

### Example of Service Delivery Organization

Service delivery organizations (SDOs) can request allocations of
prefixes for use within their SDO.  The prefix should be the name
(abbreviated where possible) of the SDO, followed by a "-" character.
The suffix is managed and allocated by the SDO, and does not need to
be added to the registry.

The suffix is text which matches the "dot-string" definition of
{{RFC7542}} Section 2.2.

For example, the 3rd Generation Partnership Project (3GPP) could
request a prefix "3gpp-", and then self-assign a suffix "baz", to
create an identifier:

> 3gpp-baz

Which then creates an NAI of the form:

> 3gpp-baz@eap.arpa

# Privacy Considerations

TBD

# Security Considerations

TBD

# Acknowledgements

TBD.

# Changelog

* 00 - initial version

--- back
