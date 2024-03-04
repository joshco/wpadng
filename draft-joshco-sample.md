---
title: "Sample"
abbrev: "wpadng"
docname: draft-joshco-wpadng-latest
category: info

ipr: trust200902
area: General
workgroup: intarea
keyword: Internet-Draft

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

A mechanism is needed to permit web clients to locate nearby web 
proxy caches. Current best practice is for end users to hand 
configure their web client (i.e., browser) with the URL of an "auto 
configuration file". In large environments this presents a 
formidable support problem.  It would be much more manageable for 
the web client software to automatically learn the configuration 
information for its web proxy settings. This is typically referred 
to as a resource discovery problem. 

Web client implementers are faced with a dizzying array of resource 
discovery protocols at varying levels of implementation and 
deployment. This complexity is hampering deployment of a "web proxy 
auto-discovery "facility.  This document proposes a pragmatic 
approach to web proxy auto-discovery.  It draws on a number of 
proposed standards in the light of practical deployment concerns. It 
proposes an escalating strategy of resource discovery attempts in 
order to find a nearby web proxy server. It attempts to provide rich 
mechanisms for supporting a complex environment, which may contain 
multiple web proxy servers. 

--- middle

# Introduction
    
The problem of locating nearby web proxy cache servers can not wait 
for the implementation and large scale deployment of various 

upcoming resource discovery protocols. The widespread success of the 
HTTP protocol and the recent popularity of streaming media has 
placed unanticipated strains on the networks of corporations, ISPs 
and backbone providers. There currently is no effective method for 
these organizations to realize the obvious benefits of web caching 
without tedious and error prone configuration by each and every end 
user. 

The de-facto mechanism for specifying a web proxy server 
configuration in web clients is the download of a script or 
configuration file named by a URL. Users are currently expected to 
hand configure this URL into their Browser or other web client.  
This mechanism suffers from a number of drawbacks: 

- Difficulty in supporting a large body of end-users. Many users 
misconfigure their proxy settings and are unable to diagnose the 
cause of their problems. 

- Lack of support for mobile clients who require a different proxy 
as their point of access changes. 

- Lack of support for complex proxy environments where there may 
exist a number of proxy servers with different affinities for 
different clients (based on network proximity, for example). 
Currently, clients would have to "know" which proxy server was 
optimal for their use. 

Currently available methods for resource discovery need to be 
exploited in the context of a well defined framework. Simple, 
functional and efficient mechanisms stand a good chance of solving 
this pressing and basic need. As new resource discovery mechanisms 
mature they can be folded into this framework with little 
difficulty. 

This document is a specification for implementers of web client 
software. It defines a protocol for automatically configuring those 
clients to use a local proxy. It also defines how an administrator 
should configure various resource discovery services in their 
network to support WPAD compatible web clients. 

While it does contain suggestions for web proxy server implementers, 
it does not make any specific demands of those parties. 


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.



--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
