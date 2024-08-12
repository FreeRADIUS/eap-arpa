----
title: The eap.arpa domain and EAP provisioning
abbrev: eap.arpa
docname: draft-ietf-emu-eap-arpa-02
updates: 9140

stand_alone: true
ipr: trust200902
area: Internet
wg: EMU Working Group
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
  org: InkBridge Networks
  email: aland@inkbridgenetworks.com

normative:
  RFC1034:
  RFC8174:
  RFC3748:
  RFC5216:
  RFC7542:
  RFC8126:
  RFC9140:

informative:
  HOTSPOT:
     title: "Passpoint"
     author:
       name: Wi-Fi Alliance
     format:
       TXT: https://www.wi-fi.org/discover-wi-fi/passpoint
  RFC2865:
  RFC7170:
  RFC8952:
  RFC9190:

venue:
  group: EMU
  mail: emut@ietf.org
  github: freeradius/eap-arpa.git

--- abstract

This document defines the eap.arpa domain as a way for EAP peers to
signal to EAP servers that they wish to obtain limited, and
unauthenticated, network access.  EAP peers signal which kind of access is required via certain pre-defined identifiers which use the Network Access Identifier (NAI) format of RFC7542.  A table of
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
exchange is not normally sent across an Authentication, Authorization,
and Accounting (AAA) proxy framework as defined in {{RFC7542}} Section
3.  Instead, the packets generally remains local to the EAP server.

This specification does not, however, forbid routing of realms in the
"eap.arpa" domain.  The use of "eap.arpa" means that any such routing
does not happen automatically.  Instead, it routing of that domain
must be explicitly configured locally, and be done with full consent
of all parties which need to authenticate NAIs in that domain.

If the EAP
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

## The realm field

The sub-domain in realm is assigned via the EAP Provisioning
Identifier Registry which is defined in [](#registry). The sub-domain
MUST follow the domain name conventions specified in {{RFC1034}}.

It is RECOMMENDED that the first sub-domain of "eap.arpa" use the EAP
method name, as defined in the IANA Extensible Authentication Protocol
(EAP) Registry, sub-table "Method Types".  However, that registry does
not follow the domain name conventions specified in {{RFC1034}}, so it
is not possible to make a "one-to-one" mapping between the Method Type
name and the subdomain.

Where it is not possible to make a direct mapping between the EAP
Method Type name (e.g. "TEAP"), and a sub-domain
(e.g. "teap.eap.arpa"), the name used in the realm registry SHOULD be
similar enough to allow the average reader to understand which EAP
Method Type is being used.

Additional sub-domains are permitted, which permit vendors and Service
delivery organizations (SDOs) the ability to self-assign a delegated
range of identifiers which cannot conflict with other identifiers.

Any realm defined in this registry (e.g. "teap.eap.arpa") also
implicitly defines a subdomain "v." (e.g. "v.teap.eap.arpa").  Vendors
can self-allocate within the "v." subdomain, using domains which they
own.  For example, 3GPP specifications could self-allocate and use the
realm "3gpp.org.v.teap.arpa".

## The username field

The username field is dependent on the EAP method being used for
provisioning. For example, {{RFC9140}} uses the username "noob". Other
EAP methods MAY omit the username as RECOMMENDED in {{RFC7542}}.  The
username of "anonymous" is NOT RECOMMENDED for specifications using
this format, even though it is permitted by {{RFC7542}}.

The username field is assigned via the EAP Provisioning Identifier
Registry which is defined in [](#registry).  The username field MAY be
empty, or else hold a fixed value. While {{RFC7542}} recommends
omitting the username portion for user privacy, the names are defined
in public specifications.  User privacy is therefore not needed here,
and the username field can be publicly visible.

## Operational Considerations

Haviong defined the format and contents of NAIs in the eap.arpa realm,
we now need to define how those NAIs are used by EAP supplicants and
EAP peers.

### EAP Supplicants

An EAP supplicant signals that it wishes a certain kind of
provisioning by using a particular NAI, along with an associated EAP
method.  The meaning of the NAI, and behavior of the supplicant are
defined by a separate specification which defines the NAI.

The NAI used by the supplicant MUST match the EAP method.  NAIs in the
eap.arpa domain MUST NOT be used for EAP methods which do not have an
entry in the "EAP Provisioning Identifiers" registry.

EAP supplicants MUST NOT allow these NAIs to be configured directly by
end users.  Instead the user (or some other process) chooses a
provisioning method, and the EAP supplicant then chooses a pre-defined NAI
which matches that provisioning method.

### EAP Peers

Implementations MUST check that the NAI matches the EAP method
being used.  Mis-matched NAIs MUST result in an EAP Failure.

For example, an NAI of "@tls.eap.arpa" MUST NOT be used for any EAP
method other than EAP-TLS.

Implementations MUST treat supplicants using a provisioning NAI as
untrusted, and untrustworthy.  Implementations MUST place supplicants
using a provisioning NAI into a limited network, such as a captive
portal.

Implementations SHOULD give supplicants access which is limited in
duration.  The provisioning process should be fairly quick, and of the
order of seconds to tens of seconds in duration.  Provisioning times
longer than that likely indicate an issue.

Implementations SHOULD give supplicants access which is limited in
scope.  The provisioning process likely does not need to download
large amounts of data, and likely does not need access to a large
number of services.  The provisioning networks SHOULD allow only
traffic which is necessary for the operation of the named provisioning
method.

Implementations SHOULD use functionality such as the RADIUS Filter-Id
attribute ({{?RFC2865, Section 5.11}}) to set packet filters for the
supplicant being provisioned.  For ease of administration, the
Filter-Id name could simply be the NAI, or a similar name.  Such
consistency aids with operational considerations when managing complex
networks.

Implementations SHOULD rate-limit provisioning attempts.  A
misbehaving supplicant should be blocked temporarily, or even
permanently. Implementations SHOULD limit the total number of
supplicants being provisioned at the same time.  There is no
requirement to allow all supplicants to connect without limit.

## Notes on AAA Routability

When we say that the eap.arpa domain is not routable in an AAA proxy
framework, we mean that the domain does not exist, and will never
resolve to anything for dynamic DNS lookups as defined in
{{?RFC7585}}.  In addition, administrators will not have statically
configuered AAA proxy routes for this domain.

In order to avoid spurious DNS lookups, RADIUS servers supporting
{{?RFC7585}} SHOULD perform filtering in the domains which are sent to
DNS.  Specifically, names in the "eap.arpa" domain SHOULD NOT be
looked up in DNS.

# Overview

For EAP-TLS, both {{RFC5216}} Section 2.1.1 and {{RFC9190}} provide
for "peer unauthenticated access".  However, those documents define no
way for a peer to signal that it is requesting such access.  The
presumption is that the peer connects with some value for the EAP
Identity, but without using a client certificate.  The EAP server is
then supposed to determine that the peer is requesting unauthenticated
access, and take the appropriate steps to limit authorization.

There appears to be no EAP peer or server implementations which
support such access, since there is no defined way to perform any of
the steps required.  i.e. to signal that this access is desired, and
then limit access.

The Wi-Fi Alliance has defined an unauthenticated EAP-TLS method,
using a vendor-specific EAP type as part of HotSpot 2.0r2 {{HOTSPOT}}.
However, there appears to be little or no deployments of this
specification.

EAP-NOOB {{RFC9140}} takes this process a step further.  It defines both
a way to signal that provisioning is desired, and also a way to
exchange provisioning information within EAP-NOOB.  That is, there is
no need for the device to obtain limited network access, as all of the
provisioning is done inside of the EAP-NOOB protocol.

TEAP {{RFC7170}} provides for provisioning via an unauthenticated TLS
tunnel.  That document provides for a server unauthenticated
provisioning mode, but the inner TLS exchange requires that both end
authenticate each other.  There are ways to provision a certificate,
but the peer must still authenticate itself to the server.

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

As the provisioning identifier is used within EAP, it necessarily has
interactions with, and effects on, the various EAP types.  This
section discusses those effects in more detail.

Some EAP methods require shared credentials such as passwords in order
to succeed.  For example, both EAP-MSCHAPv2 (PEAP) and EAP-PWD
{{?RFC5931}} perform cryptographic exchanges where both parties
knowing a shared password.  Where password-based methods are used, the
password MUST be the same as the provisioning identifier.

This requirement also applies to TLS-based EAP methods such as TTLS
and PEAP.  Where the TLS-based EAP method provides for an inner
identity and inner authentication method, the credentials used there
MUST be the provisioning identifier for both the inner identity, and
any inner password.

It is RECOMMENDED that provisioning be done via a TLS-based EAP methods.
TLS provides for authentication of the EAP server, along with security
and confidentiality of any provisioning data exchanged in the tunnel.
Similarly if provisioning is done in a captive portal outside of EAP,
EAP-TLS permits the EAP peer to run a full EAP authentication session
while having nothing more than a few certification authorities (CAs)
locally configured.

## EAP-TLS

This document defines an identifier "portal@tls.eap.arpa", which is
the first step towards permitted unauthenticated client provisioning
in EAP-TLS.  The purpose of the identifier is to allow EAP peers to
signal EAP servers that they wish to obtain a "captive portal" style
network access.

This identifier signals the EAP server that the peer wishes to obtain
"peer unauthenticated access" as per {{RFC5216}} Section 2.1.1 and
{{RFC9190}}.

An EAP server which agrees to authenticate this request MUST ensure
that the device is placed into a captive portal with limited network
access.  Implementations SHOULD limit both the total amount of data
transferred by devices in the captive portal, and SHOULD also limit the
total amount of time a device spends within the captive portal.

Further details of the captive portal architecture can be found in
{{RFC8952}}.

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

It is possible to also use TLS-PSK with EAP-TLS for this provisioning.
In which case, the PSK identity MUST the same as the EAP Identifier,
and the PSK MUST be the provisioning identifier.

## TLS-based EAP methods

Other TLS-based EAP methods such as TTLS and PEAP can use the same
method as defined for EAP-TLS above.  The only difference is that the
inner identity and password is also the provisioning identifier.

Is is RECOMMENDED that provisioning methods use EAP-TLS in preference
to any other TLS-based EAP methods.  As the credentials for other
methods are predefined and known in advance, those methods offer little
benefit over EAP-TLS.

## EAP-NOOB

It is RECOMMENDED that server implementations of EAP-NOOB accept both
identities "noob@eap-noob.arpa" and "@noob.eap.arpa" as synonyms.

It is RECOMMENDED that EAP-NOOB peers use "@noob.eap.arpa" first, and
if that does not succeed, use "noob@eap-noob.arpa"

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

### Domain Name Reservation Considerations

This section answers the questions which are required by Section 5 of {{?RFC6761}}.  At a high level, these new domain names are used in certain situations in EAP.  The domain names are never seen by users, and they do not appear in any networking protocol other than EAP.

1. Users:

> User are not expected to recognise these names as special or use them differently from other domain names.  The use of these names in EAP is invisible to end users.

2. Application Software:

> EAP servers and clients are expected to make their software recognize these names as special and treat them differently.  This document discusses that behavor.
>
> EAP supplicants should recognize these names as special, and should refuse to allow users to enter them in any interface.
>
> EAP servers and RADIUS servers should recognize the ".arpa" domain as special, and refuse to do dynamic DNS lookups ({{?RFC7585}}) for it.

3. Name Resolution APIs and Libraries:

> Writers of these APIs and libraries are not expected to recognize these names or treat them differently.

4. Caching DNS Servers:

> Writers of caching DNS servers are not expected to recognize these names or treat them differently.

5. Authoritative DNS Servers:

> Writers of authoritative DNS servers are not expected to recognize these names or treat them differently.

6.  DNS Server Operators:

> These domain names have minimal impact on DNS server operators.  They should never be used in DNS, or in any networking protocol outside of EAP.
>
> Some DNS servers may receive lookups for this domain, if EAP or RADIUS servers are configured to do dynamic DNS lookups for realms as defined in {{?RFC7585}}, and where those servers are not updated to ignore the ".arpa" domain.  When queried for the "eap.arpa" domain, DNS servers SHOULD return an NXDOMAIN error.
>
> If they try to configure their authoritative DNS as authoritative for this reserved name, compliant name servers do not need to do anything special.  They can accept the domain or reject it.  Either behavior will have no impact on this specification.

7. DNS Registries/Registrars:

> DNS Registries/Registrars should deny requests to register this reserved domain name.

## EAP Provisioning Identifiers Registry {#registry}

IANA is instructed to add the following new registry to the "Extensible Authentication Protocol (EAP) Registry" group.

Assignments in this registry are done via "Expert Review" as described in {{RFC8126}} Section 4.5.  Guidelines for experts is provided in [](#guidelines).

The contents of the registry are as follows.

Title

> EAP Provisioning Identifiers

Registration Procedure(s)

> Expert review

Reference

> THIS-DOCUMENT

Registry

> NAI
>
>> The Network Access Identifier in {{RFC7542}} format.  Names are lowercase, and are listed in sorted order.
>
> Method Type
>
>> The EAP method name, taken from the "Description" field of the EAP "Method Types" registry.
>
> Reference
>
>> Reference where this identifier was defined.

### Initial Values

The following table gives the initial values for this table.

NAI,Method-Type,Description,Reference

@noob.eap.arpa,EAP-NOOB,RFC9140 and THIS-DOCUMENT
portal@tls.eap.arpa,EAP-TLS,RFC9190 and THIS-DOCUMENT

## Guidelines for Designated Experts {#guidelines}

The following text gives guidelines for Designated Experts who review
allocation requests for this registry.

### NAIs

The intent is for the NAI to contain both a reference to the EAP
Method Type, and a description of the purpose of the NAI.  For
example, with an EAP Method Type "name", and a purpose "action", the
NAI SHOULD be of the form "action@foo.eap.arpa".

The NAI MUST satisfy the requirements of the {{RFC7542, Section 2.2}}
format.  The utf8-username portion MAY be empty.  The utf8-username
portion MUST NOT be "anonymous".  The NAI MUST end with "eap.arpa".
The NAI SHOULD NOT contain more than one subdomain.

The sub-domain of the NAI field should correspond to the EAP Method
Type name.  Care should be taken so that the domain name conventions
specified in {{RFC1034}} are followed.

The NAIs in this registry are case-insensitive.  While {{RFC7542}}
notes that similar identifiers of different case can be considered to
be different, for simplicity this registry requires that all entries
MUST be lowercase.

Identifiers MUST be unique when compared in a case-insensitive
fashion.  While {{RFC7542}} notes that similar identifiers of
different case can be considered to be different, this registry is
made simpler by requiring case-insensitivity.

Entries in the registry shuld be short.  NAIs defined here will
generally be sent in a RADIUS packet in the User-Name attribute
({{RFC2865}} Section 5.1).  That specification recommends that
implementations should support User-Names of at least 63 octets.  NAI
length considerations are further discussed in {{RFC7542}} Section
2.3, and any allocations in this registry needs to take those
limitations into consideration.

Implementations are likely to support a total NAI length of 63 octets.
Lengths between 63 and 253 octets may work.  Lengths of 254 octets or
more will not work with RADIUS {{RFC2865}}.

## Method Type

Values in "Method Type" field of this registry MUST be taken from the
IANA EAP Method Types registry or else it MUST be an Expanded Type
which usually indicates a vendor specific EAP method.

The EAP Method Type MUST provide an MSK and EMSK as defined in
{{RFC3748}}.  Failure to provide these keys means that the method will
not be usable within an authentication framework which requires those
methods, such as with IEEE 802.1X.

## Designated Experts

For registration requests where a Designated Expert should be
consulted, the responsible IESG area director should appoint the
Designated Expert.  But in order to allow for the allocation of values
prior to the RFC being approved for publication, the Designated Expert
can approve allocations once it seems clear that an RFC will be
published.

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

## Vendor Self Assignment

This registry provides for vendor self-assignment within the NAI
space.  Any NAI defined in this registry also implicitly defines a
subdomain "v." for vendor or SDO self-allocation.  For example, an NAI
"action@foo.eap.arpa" uses a realm "foo.eap.arpa".  Vendors and SDOs
can self-allocate in this space, under the "v." subdomain,
"v.foo.eap.arpa".

Any domain name which is registered as a Fully Qualified Domain Name
(FQDN) within the DNS can use that name within the "v." subdomain.
For example, 3GPP specifications could self-allocate the realm
""@3gpp.org.v.foo.arpa", and then use any utf8-username within that
realm.

Note that the right to use a self-allocated name is tied to the
ownership of the corresponding subdomain.  If an entity loses the
right to use a domain name, the right to use that domain name within
the "v." self-allocation space is immediately revoked.

# Privacy Considerations

The EAP Identity field is generally publicly visible to parties who
can observe the EAP traffic.  As the names given here are in a public
specification, there is no privacy implication to exposing those names
within EAP.  The entire goal of this specification is in fact to make
those names public, so that unknown (and private) parties can publicly
(and anonymously) declare what kind of network access they desire.

However, there are many additional privacy concerns around this
specification.  Most EAP traffic is sent over RADIUS {{RFC2865}}.  The
RADIUS Access-Request packets typically contain large amounts of
information such as MAC addresses, device location, etc.

This specification does not change RADIUS or EAP, and as such does not
change which information is publicly available, or is kept private.
Those issues are dealt with in other specifications.

However, this specification can increase privacy by allowing devices
to anonymously obtain network access, and then securely obtain
credentials.

# Security Considerations

This specification defines a framework which permits unknown,
anonymous, and unauthenticated devices to request and to obtain
network access.  As such, it is critical that network operators
provide limited access to those devices.

Future specifications which define an NAI within this registry, should
give detailed descriptions of what kind of network access is to be
provided.

## On-Path Attackers and Impersonation

In most EAP use-cases, the server identity is validated (usually
through a certificate), or the EAP method allows the TLS tunnel to be
cryptographically bound to the inner application data.  For the
methods outlined here, the use of public credentials, and/or skipping
server validation allows "on-path" attacks to succeed where they would
normally fail

EAP supplicants and peers MUST assume that all data sent over an EAP
session is visible to attackers, and can be modified by them.

The methods defined here SHOULD only be used to bootstrap initial
network access.  All subsequence application-layer traffic SHOULD be
full authenticated and secured with systems such as IPSec or TLS.

## Provisioning is Unauthenticated

We note that this specification allows for unauthenticated supplicants
to obtain network access, however limited.  As with any
unauthenticated process, it can be abused.  Implementations should
take care to limit the use of the provisioning network.

## Privacy Considerations

The NAIs used here are contained in a public registry, and therefore
do not have to follow the username privacy recommendations of
{{RFC7542, Section 2.4}}.  However, there may be other personally
identifying information contained in EAP or AAA packets.  This
situation is no different from normal EAP authentication, and thus
has no additional positive or negative implications for privacy.

# Acknowledgements

Mohit Sethi provided valuable insight that using subdomains was better
and more informative than the original method, which used only the
utf8-username portion of the NAI.

# Changelog

* 00 - initial version

--- back
