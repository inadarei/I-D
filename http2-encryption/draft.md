---
title: Opportunistic Encryption for HTTP URIs
abbrev: Opportunistic HTTP Encryption
docname: draft-nottingham-http2-encryption-03
date: 2014
category: std

ipr: trust200902
area: General
workgroup:
keyword: Internet-Draft

stand_alone: yes
pi: [toc, tocindent, sortrefs, symrefs, strict, compact, comments, inline]

author:
 -
    ins: M. Nottingham
    name: Mark Nottingham
    organization:
    email: mnot@mnot.net
    uri: http://www.mnot.net/
 -
    ins: M. Thomson
    name: Martin Thomson
    organization: Mozilla
    email: martin.thomson@gmail.com

normative:
  RFC2119:
  RFC2818:
  RFC5246:
  I-D.ietf-httpbis-http2:
  I-D.ietf-httpbis-alt-svc:
  I-D.ietf-httpbis-p6-cache:
  I-D.ietf-websec-key-pinning:

informative:
  firesheep:
    target: http://codebutler.com/firesheep/
    title: Firesheep
    author:
      ins: E. Butler
      name: Eric Butler
    date: 2010
  streetview:
    target: http://www.wired.com/threatlevel/2012/05/google-wifi-fcc-investigation/
    title: The Anatomy of Google's Wi-Fi Sniffing Debacle
    author:
      ins: D. Kravets
      name: David Kravets
      organization: Wired
    date: 2012
  xkeyscore:
    target: http://www.theguardian.com/world/2013/jul/31/nsa-top-secret-program-online-data
    title: NSA tool collects 'nearly everything a user does on the internet'
    author:
      ins: G. Greenwald
      name: Glenn Greenwald
      organization: The Guardian
    date: 2013
  I-D.mbelshe-httpbis-spdy:
  RFC2804:
  RFC3365:
  RFC6454:
  RFC6973:


--- abstract

This describes how "http" URIs can be resolved using Transport Layer Security
(TLS).

--- middle

# Introduction

This document describes a use of the "alternate services" layer described in
{{I-D.ietf-httpbis-alt-svc}} to decouple the URI scheme from the use and
configuration of underlying encryption, allowing a "http" URI to be upgraded to
use TLS opportunistically.

Using TLS requires acquiring and configuring a valid certificate, which means
that some deployments find supporting TLS difficult. Therefore, this document
describes a usage model whereby sites that serve "http" URIs over TLS are not
required to support strong server authentication.

A mechanism for limiting the potential for active attack is described in {{http-tls}}. This
provides clients with additional protection against active attack for a period
after successfully connecting to a server using TLS.  This does not offer the
same level of protection as afforded to "https" URIs, but increases the
likelihood that an active attack be detected.

## Goals and Non-Goals

The immediate goal is to make the use of HTTP more robust in the face of passive
monitoring.

Such passive attacks are often opportunistic; they rely on sensitive
information being available in the clear. Furthermore, they are often broad,
where all available data is collected en masse, being analyzed separately for
relevant information.

A mechanism for limiting the potential for active attack is described. This
provides clients with additional protection against active attack for a period
after successfully connecting to a server using TLS.  This does not offer the
same level of protection as afforded to "https" URIs, but increases the
likelihood that an active attack be detected.

A secondary, but significant, goal is to provide for ease of implementation,
deployment and operation.  This mechanism should have a minimal impact upon
performance.

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in {{RFC2119}}.

# Using HTTP over TLS

A server that supports the resolution of HTTP URIs can provide an alternative
service advertisement {{I-D.ietf-httpbis-alt-svc}} for a protocol that uses TLS,
such as "h2" {{I-D.ietf-httpbis-http2}}, or "http/1.1" {{RFC2818}}.

A client that sees this alternative service advertisement can direct future
requests for the associated origin to the identified service.

A client that places the importance of passive protections over performance
might choose to send no further requests over cleartext connections if it
detects the alternative service advertisement.  If the alternative service
cannot be successfully connected, the client might resume its use of the
cleartext connection.

A client can also explicitly probe for an alternative service advertisement by
sending a request that bears little or no sensitive information, such as one
with the OPTIONS method.  Clients with expired alternative services information
could make a similar request in parallel to an attempt to contact an alternative
service, to minimize the delays that might be incurred by failing to contact the
alternative service.

# Server Authentication

There are no expectations with respect to security when it comes to resolving
HTTP URIs.  Server authentication, as described in {{RFC2818}}, creates a number
of operational challenges.  For these reasons, server authentication is not
mandatory for HTTP URIs.

When connecting to a service, clients do not perform the server authentication
procedure described in Section 3.1 of {{RFC2818}}.  The server certificate, if
one is proffered, is not checked for validity, expiration, issuance by a trusted
certificate authority or matched against the name in the URI.  A server is
therefore able to provide any certificate, or even select TLS cipher suites that
do not include authentication.

A client MAY perform additional checks on the certificate that it is offered (if
the server does not select an unauthenticated TLS cipher suite).  For instance,
a client could examine the certificate to see if it has changed over time.

In order to retain the authority properties of "http" URIs, and as stipulated by
{{I-D.ietf-httpbis-alt-svc}}, clients MUST NOT use alternative services that
identify a different host, unless the alternative service indication is
authenticated.  This is not currently possible for "http" URIs on cleartext
transports.

# Interaction with "https" URIs

A service that is discovered to support "http" URIs might concurrently support
"https" URIs.  HTTP/2 permits the sending of requests for multiple origins (see
{{RFC6454}}) on the one connection.  When using alternative services, both HTTP
and HTTPS URIs might be sent on the same connection.

HTTPS URIs rely on server authentication.  Therefore, if a connection is
initially created without authenticating the server, requests for HTTPS
resources cannot be sent over that connection until the server certificate is
successfully authenticated.  Section 3.1 of {{RFC2818}} describes the basic
mechanism, though the authentication considerations in
{{I-D.ietf-httpbis-alt-svc}} could also apply.

Connections that are established without any means of server authentication (for
instance, the purely anonymous TLS cipher suites), cannot be used for "https"
URIs.

# Persisting use of TLS {#http-tls}

Note: this is a very rough take on an approach that would provide a limited form
of protection against downgrade attack.  It's unclear at this point whether the
additional effort (and modest operational cost), is worthwhile.

Two factors ensure that active attacks are trivial to mount:

- A client that doesn't perform authentication an easy victim of server
  impersonation, through man-in-the-middle attacks.

- A client that is willing to use cleartext to resolve the resource will do so
  if access to any TLS-enabled alternative services is blocked at the network
  layer.

Given that the primary goal of this work is to prevent passive attacks, these
are of less concern than they might otherwise be, but a modest form of
protection against these attacks can be provided for clients on return visits to
a server.

A server can make a commitment to providing service over TLS in future requests.
This allows clients to detect an active attack and fail requests when the server
cannot be contacted using TLS.

The drawback with this approach is that servers can only make this commitment if
they are strong authenticated. Otherwise, server impersonation could be used to
create a persistent denial of service.

## The HTTP-TLS Header Field

A server makes this commitment by sending a `HTTP-TLS` header field:

    HTTP-TLS     = 1#parameter

For example:

    HTTP/1.1 200 OK
    Content-Type: text/html
    Cache-Control: 600
    Age: 30
    Date: Thu, 1 May 2014 16:20:09 GMT
    HTTP-TLS: ma=3600

A client that has has not authenticated the server MAY do so when it sees a
`HTTP-TLS` header field.  The server is authenticated as described in Section
3.1 of {{RFC2818}}, noting the additional requirements in
{{I-D.ietf-httpbis-alt-svc}}.  If server authentication is successful, the
client can persistently store a record that the requested origin {{RFC6454}} can
be retrieved over TLS.

Persisted information expires after a period determined by the value of the "ma"
parameter.  See Section 4.2.3 of {{I-D.ietf-httpbis-p6-cache}} for details of
determining response age.

    ma-parameter     = delta-seconds

Requests for an origin that has a persisted, unexpired value for `HTTP-TLS` MUST
fail if they cannot be made over an authenticated TLS connection.

## Operational Considerations

To avoid situations where a persisted value of `HTTP-TLS` causes a client to be
unable to contact a site, clients SHOULD limit the time that a value is
persisted for a given origin.  A hard limit might be set to a month.  A lower
limit might be appropriate for initial observations of `HTTP-TLS`; the certainty
that a site has set a correct value - and the corresponding limit on persistence
- can increase as the value is seen more over time.

Once a server has indicated that it will support authenticated TLS, a client MAY
use key pinning {{I-D.ietf-websec-key-pinning}} or any other mechanism that
would otherwise be restricted to use with HTTPS URIs, provided that the
mechanism can be restricted to a single HTTP origin.


# Security Considerations

## Indicators

User Agents MUST NOT provide any special security indicia when an "http"
resource is acquired using TLS.  In particular, indicators that might suggest
the same level of security as "https" MUST NOT be used (e.g., using a "lock
device").


## Downgrade Attacks {#downgrade}

A downgrade attack against the negotiation for TLS is possible.  With the
`HTTP-TLS` header field, this is limited to occasions where clients have no
prior information (see {{privacy}}), or when persisted commitments have
expired.

For example, because the `Alt-Svc` header field {{I-D.ietf-httpbis-alt-svc}}
likely appears in an unauthenticated and unencrypted channel, it is subject to
downgrade by network attackers.  In its simplest form, an attacker that wants
the connection to remain in the clear need only strip the `Alt-Svc` header field
from responses.

As long as a client is willing to use cleartext TCP to contact a server, these
attacks are possible.  The `HTTP-TLS` header field provides an imperfect
mechanism for establishing a commitment.  The advantage is that this only works
if a previous connection is established where an active attacker was not
present.  A continuously present active attacker can either prevent the client
from ever using TLS, or offer a self-signed certificate.  This would prevent the
client from ever seeing the `HTTP-TLS` header field, or if the header field is
seen, from successfully validating and persisting it.

## Privacy Considerations {#privacy}

Clients that persist state for origins can be tracked over time based on their
use of this information.  Persisted information can be cleared to reduce the
ability of servers to track clients.  A browser client MUST clear persisted all
alternative service information when clearing other origin-based state (i.e.,
cookies).


--- back

# Acknowledgements

Thanks to Patrick McManus, Eliot Lear, Stephen Farrell, Guy Podjarny, Stephen
Ludin, Erik Nygren, Paul Hoffman, Adam Langley, Eric Rescorla and Richard Barnes
for their feedback and suggestions.


# Recent History and Background

One of the design goals for SPDY {{I-D.mbelshe-httpbis-spdy}} was increasing
the use of encryption on the Web, achieved by only supporting the protocol over
a connection protected by TLS {{RFC5246}}.

This was done, in part, because sensitive information -- including not only
login credentials, but also personally identifying information (PII) and even
patterns of access -- are increasingly prevalent on the Web, being evident in
potentially every HTTP request made.

Attacks such as FireSheep {{firesheep}} showed how easy it is to gather such
information when it is sent in the clear, and incidents such as Google's
collection of unencrypted data by its StreetView Cars {{streetview}} further
illustrated the risks.

In adopting SPDY as the basis of HTTP/2 {{I-D.ietf-httpbis-http2}}, the HTTPbis
Working Group agreed not to make TLS mandatory to implement (MtI) or mandatory
to use (MtU) in our charter, despite an IETF policy to prefer the "best
security available" {{RFC3365}}.

There were a variety of reasons for this, but most significantly, HTTP is used
for much more than the traditional browsing case, and encryption is not needed
for all of these uses. Making encryption MtU or MtI was seen as unlikely to
succeed because of the wide deployment of HTTP URIs.

However, since making that decision, there have been developments that
have caused the Working Group to discuss these issues again:

1. Active contributors to some browser implementations have stated that their
products will not use HTTP/2 over unencrypted connections. If this eventuates,
it will prevent wide deployment of the new protocol (i.e., it couldn't be used
with those products for HTTP URIs; only HTTPS URIs).

2. It has been reported that surveillance of HTTP traffic takes place on a broad
scale {{xkeyscore}}. While the IETF does not take a formal, moral position on
wiretapping, we do have a strongly held belief "that both commercial development
of the Internet and adequate privacy for its users against illegal intrusion
requires the wide availability of strong cryptographic technology"
{{RFC2804}}. This requirement for privacy is further reinforced by {{RFC6973}}.

As a result, we decided to revisit the issue of how encryption is used in HTTP/2
at IETF87.


# Frequently Asked Questions

## Will this make encryption mandatory in HTTP/2?

Not in the sense that this proposal would have it required (with a MUST) in the
specification.

What might happen, however, is that some browser implementers will take the
flexibility that this approach grants and decide to not negotiate for HTTP/2
without one of the encryption profiles. That means that servers would need to
implement one of the encryption-enabling profiles to interoperate using HTTP/2
for HTTP URIs.

## No certificate checks? Really?

Since TLS isn't in use for any "http" URIs today, there is no net loss of
security, and we gain some privacy from passive attacks.

This makes TLS significantly simpler to deploy for servers; they are able to use
a self-signed certificate.

With the `HTTP-TLS` header field, we are able to gain a measure of protection.

## Why do this if a downgrade attack is so easy?

There are many attack scenarios (e.g., third parties in coffee shops) where
active attacks are not feasible, or much more difficult.

Additionally, active attacks can often be detected, because they change
protocol interactions; as such, they bring a risk of discovery.