---
title: "Web Proxy Auto Discovery Next Generation"
abbrev: "wpadng"
docname: draft-joshco-wpadng-latest
category: info

ipr: trust200902
area: General
keyword: Internet-Draft
stream: independent

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: J. Cohen
    name: Josh Cohen
    organization: Self
    email: joshco@gmail.com

normative:

informative:


--- abstract

This specification aims to modernize Web Proxy Automatic Discovery ({{!WPAD=I-D.ietf-wrec-wpad-01}}) which was defined in 1997.  At that time, the World Wide Web was much
earlier in its evolution. This specification provides more modern discovery mechanisms and incorporates {{!PVD=I-D.draft-pauly-intarea-proxy-config-pvd-02}}


--- middle

# Introduction

Web Proxy Automatic Discovery Next Generation (WPADNG) defines a mechanism for Web Browsers, applications, or operating system services to discover nearby Web Proxy Severs.

The Web Proxy Auto-Discovery (WPADNG) problem reduces to
providing the web client a mechanism for discovering the URL of the Configuration File (CFILE). Once this Configuration File URL (CURL) is known, the client software already contains mechanisms for retrieving and interpreting the CFILE to enable access to the
specified proxy cache servers.

The goal of the wpadng process is to discover the correct CURL at which to retrieve the CFILE. The client
is *not* trying to directly discover the name of the proxy server.
That would circumvent the additional capabilities provided by proxy
Configuration Files (such as load balancing, request routing to an
array of servers, automated fail-over to backup proxy server.)


Different clients requesting the CURL may
receive completely different CFILEs in response. The web server may
send back different CFILES based on a number of criteria such as the
"User-Agent" header, "Accept" headers, client IP address/subnet,
etc.  The same client could conceivably receive a different CFILE on
successive retrievals (as a method of round-robin load balancing,
for example).

This document will discuss a set of mechanisms for discovering the
Configuration URL. The client will attempt them in a predefined
order, until one succeeds.

# CURLs and URNs

When a CURL is returned by discovery, it SHOULD be either a URL which is retrievable or a URN.

This specification defines the following URN for use with WPADNG.

`urn:wpadng:none`

When this URN is returned as the CURL, the meaning is that the current network has no proxy server, and the client SHOULD stop any further discovery.  This can be helpful in avoiding unnecessary network round trips and  preventing discovery of rogue proxy servers.

# The Discovery Process

## WPADNG Overview

WPADNG uses a collection of pre-existing Internet resource discovery mechanisms to perform web proxy auto-discovery.  The WPADNG protocol specifies the following:

- how to use each mechanism for the specific purpose of web proxy
    auto-discovery
- the order in which the mechanisms should be performed
- the minimal set of mechanisms which must be attempted by a WPADNG
    compliant web client

The resource discovery mechanisms utilized by WPADNG, in order, are as follows.

- Dynamic Host Configuration Protocol v4 or v6 depending on network.
- DNS-SD

Clients only attempt mechanisms that they support. Each
time the discovery attempt succeeds; the client uses the information
obtained to construct a CURL. If the CURL is the URN `urn:wpadng:none` or if a CFILE is successfully retrieved, the process is complete.

If not, the client resumes where it left of in the predefined series of resource discovery requests. If no untried mechanisms remain and a CFILE has not been successfully retrieved, the WPADNG protocol fails and the client is configured to use no proxy server.

First the client tries DHCP, followed by DNSSD. If no CFILE has been
retrieved the client will retry the DNSSD mechanism multiple times. Each time through the QNAME being
used in the DNSSD query is made less and less specific. In this manner
the client can locate the most specific configuration information
possible, but can fall back on less specific information. Every DNSSD
lookup has the QNAME prefixed with “_wpadng._tcp” to indicate the resource type being requested.

As an example, consider a client with hostname johns-
desktop.development.example.com. Assume the web client software supports all of the mechanisms listed above. This is the sequence of
discovery attempts the client would perform until one succeeded in
locating a valid CFILE:

- DNSSD PTR lookup on QNAME=_wpadng._tcp.development.example.com.
- DNSSD PTR lookup on QNAME=_wpadng._tcp.example.com.

## When to Execute WPADNG

Web clients need to perform the WPADNG protocol periodically to
maintain correct proxy settings. This should occur on a regular
basis corresponding to initialization of the client software or the
networking stack below the client. As well, WPADNG will need to occur
in response to expiration of existing configuration data.  The
following sections describe the details of these scenarios.

### Periodic Discovery

The web proxy auto-discovery process MUST occur at least as
frequently as one of the following two options. A web client can use
either option depending on which makes sense in their environment.
Clients MUST use at least one of the following options. They MAY
also choose to implement both options.
- Upon startup of the web client.
- Whenever there indication from the networking stack that the  IP
address of the client host either has, or could have, changed.

In addition, the client MUST attempt a discovery cycle upon
expiration of a previously downloaded CFILE in accordance with
HTTP/1.1.

###    Upon Startup of the Web Client

For many types of web client (like web browsers) there can be many
instances of the client operating for a given user at one time. This
is often to allow display of multiple web pages in different
windows, for example. There is no need to re-perform WPADNG every time
a new instance of the web client is opened. WPADNG MUST be performed
when the number of web client instances transitions from 0 to 1. It
SHOULD NOT be performed as additional instances are created.

###    Network Stack Events

Another option for clients is to tie the execution of WPADNG to
changes in the networking environment. If the client can learn about
the change of the local host’s IP address, or the possible change of
the IP address, it MUST re-perform the WPADNG protocol.  Many
operating systems provide indications of “network up” events, for
example. Those types of events and system-boot events might be the
triggers for WPADNG in many environments.

###    Expiration of the CFILE

The HTTP retrieval of the CURL may return HTTP headers specifying a
valid lifetime for the CFILE returned. The client MUST obey these
timeouts and rerun the PAD process when it expires. A client MAY
rerun the WPADNG process if it detects a failure of the currently
configured proxy (which is not otherwise recoverable via the
inherent mechanisms provided by the currently active Configuration
File).

Whenever the client decides to invalidate the current CURL or CFILE,
it MUST rerun the entire WPADNG protocol to ensure it discovers the
currently correct CURL. Specifically, if the valid lifetime of the
CFILE ends(as specified by the HTTP headers provided when it was
retrieved),the complete WPADNG protocol MUST be rerun. The client MUST
NOT simply re-use the existing CURL to obtain a fresh copy of the
CFILE.

A number of network round trips, broadcast and/or multicast
communications may be required during the WPADNG protocol. The WPADNG
protocol SHOULD NOT be invoked at a more frequent rate than
specified above (such as per-URL retrieval).

## Discovery Mechanisms

Each of the resource discovery methods will be marked as to whether
the client MUST, SHOULD, MAY, or MUST NOT implement them to be
compliant. Client implementers are encouraged to implement as many
mechanisms as possible, to promote maximum interoperability.


| Discovery Mechanism     | Status     |
| -------------------     | ------     |
| DHCP                    | MUST       |
| DNS-SD                  | MUST       |

### DNS Serice Discovery (DNS-SD)

Client implementations MUST support {{!DNSSD=RFC6763}} discovery of
proxy configuration files.  To suport this, a DNS server advertising
WPADNG will have the following resource records:

SHOULD advertise PTR records which may return multiple advertised service instances
or a single instance.

MUST advertise TXT records conformant with {{DNSSD}}

SHOULD advertise SRV records conformant with {{DNSSD}}

#### PTR Record Definition

For example purposes, we assume a client has attached to the
enterprise network at Example Coporation, which uses the dommain `example.org`

WPADNG PTR records should have the following naming scheme:

~~~
_wpadng._tcp.example.org
~~~

When a client queries for the PTR record, the DNS server replies
will contain zero, one or more responses.  These responses contain WPADNG
instance names, and priorities.

| Instance Name                       |
| -------------                       |
| primary._wpadng._tcp.example.org    |
| secondary._wpadng._tcp.example.org  |
| _wpadng._tcp.example.org            |

In the above example. the enterprise advertises instance names for proxy servers
This allows an enterprise to provide redundancy and failover.

#### TXT Record Definition

WPADNG DNS TXT records MUST have the following format

~~~
txtvers=1
url=CURL
~~~

The CURL value is the URL to retrieve a Proxy Configuration FIle.


#### DNSSD Client behavior

When attempting to discover web proxy servers via {{DNSSD}}, the following sequence should be used:

1. Query PTR records
2. Query TXT records
3. Compose Candidate URL
4. Retreive configuration file

### DHCP v4

Client implementations MUST support DHCP. DHCP has widespread
support in numerous vendor hardware and software implementations, and
is widely deployed. It is also perfectly suited to this task, and is
used to discover other network resources (such a time servers,
printers, etc). The DHCP protocol is detailed in {{!DHCP=RFC2131}}.
We propose a new DHCP option with code 252 for use in web proxy
auto-discovery. See {{!DHCPOPT=RFC2132}} for a list of existing DHCP
options.

The client should obtain the value of the DHCP option code 252 as
returned by the DHCP server. If the client has already conducted
DHCP protocol during its initialization, the DHCP server may already
have supplied that value. If the value is not available through a
client OS API, the client SHOULD use a DHCPINFORM message to query
the DHCP server to obtain the value.

The DHCP option code for WPAD is 252 by agreement of the DHC working
group chair.  This option is of type STRING.  This string contains a
URL which points to an appropriate config file.  The STRING is of
arbitrary size.

Example values:

~~~
http://server.domain/proxyconfig.pac
urn:wpadng:none
~~~

### DHCP v6 (TBD)


### Timeouts

Implementers are encouraged to limit the time elapsed in each
discovery phase.  When possible, limiting each phase to 10 seconds
is considered reasonable.  Implementers may choose a different value
which is more appropriate to their network properties.  For example,
a device implementation, which operated over a wireless network, may
use a much larger timeout to account for low bandwidth or high
latency.

## Retrieving the CFILE at the CURL

The client then requests the CURL via HTTP.

### Content Negotiation for PVD

When making the request it MUST transmit HTTP "Accept" headers
indicating what CFILE formats it is capable of accepting.

For PVD, the Accept header value is `application/pvd_json`

For existing WPAD clients, the most commonly used format is Netscape's
Proxy Auto Config (PAC) file, the value is 'application/x-ns-proxy-autoconfig'

Clients that support PVD can use content negotiation to prefer PVD,
while accepting PAC as a fallback.

~~~
:method = GET
:authority = HOST
:path = PATH
accept = application/pvd+json
accept = application/x-ns-proxy-autoconfig
~~~

Clients that do not support PVD will continue to use PAC, and not include
PVD in the accept header.

When receiving a GET at the CURL, a proxy server can decide wether to return
PVD or PAC, providing backward compatibility as well as an upgrade path.

The client MUST follow HTTP redirect directives (response codes 3xx)
returned by the server. The client SHOULD send a valid "User-Agent"
header.


## Resuming Discovery

If the HTTP request fails for any reason (fails to connect, server
error response, etc) the client MUST resume the search for a
successful CURL where it left off. It should continue attempting
other sub-steps in a specific discovery mechanism, and then move on
to the next mechanism or TGTDOM iteration, etc.

# Proxy Server Considerations

As mentioned in the previous section, it is suggested that proxy
servers be capable of acting as a web server, so that they can host
the CURL directly.

The implementers of proxy servers are most likely to understand the
deployment situations of proxy caches, the formats of proxy
configuration files, etc. They can also build in the ability select
a CFILE based on all the various inputs at the time of the CURL
request("User-Agent", "Accept", client IP address/subnet/hostname,
topological distribution of nearby proxy servers, etc).


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Security Considerations

This document does not address security of the protocols involved.
The WPAD protocol is vulnerable to existing identified weaknesses in
DHCP and DNS. The groups driving those standards, as well as the SLP
protocol standards, are addressing security.

When using DHCP discovery, clients are encouraged to use unicast
DHCP INFORM queries instead of broadcast queries which are more
easily spoofed in insecure networks.

Minimally, it can be said that the WPAD protocol does not create new
security weaknesses.


# IANA Considerations

## DHCPv6 Option Code

This document requests a new DHCPv6 Option.

## DHCPv4 Option Code

This document seeks to register DHCPv4 option code 252.

--- back

# Acknowledgments

## Acknowledgements for current versions

The editor(s) of this specification would like acknowledge the contributions
of the previous authors.

   Paul Gauthier
   Inktomi Corporation

   Martin Dunsmuir
   RealNetworks, Inc.

   Charles Perkins
   Sun Microsystems, Inc.

## Acknowledgements from previous versions

The authors' work on this specification would be incomplete without
the assistance of many people.  Specifically, the authors would like
the express their gratitude to the following people:

Chuck Neerdaels, Inktomi, for providing assistance in the design of
the WPADNG protocol as well as for providing reference
implementations.

Arthur Bierer, Darren Mitchell, Sean Edmison, Mario Rodriguez, Danpo
Zhang, and Yaron Goland, Microsoft, for providing implementation
insights as well as testing and deployment.

Ari Luotonen, Netscape, for his role in designing the first web
proxy.

In addition, the authors are grateful for the feedback provided by
the following people:

* Jeremy Worley - RealNetworks
* Eric Twitchell - United Parcel Service
