# The eap.arpa domain and EAP provisioning

This document gives implementation and operational considerations for using the eap.arpa domain.

## Provisioning

EAP-TLS [RFC 5216](https://datatracker.ietf.org/doc/html/rfc5216) allows for "peer unauthenticated access", where a system can gain network access without providing credentials.  However, it does not define how that works, and there are still open questions:

* how does the peer signal the server that it wants unauthenticated access?

* what network is the user placed into?

This document addresses that need.  It defines the `eap.arpa` domain as being used solely for provisioning.  The type of provisioning is determined by the "username" portion of the NAI, as defined in [RFC 7542](https://datatracker.ietf.org/doc/html/rfc7542)

This document is for the IETF EMU WG
http://datatracker.ietf.org/wg/emu
