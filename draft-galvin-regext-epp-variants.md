---
stand_alone: true
ipr: trust200902
cat: std
submissiontype: IETF
area: Applications
wg: regext

docname: draft-galvin-regext-epp-variants-02

title: Domain variant support for EPP
abbrev: EPP Variants
lang: en
kw:
  - EPP
author:
- name: James Galvin
  org: Identity Digital
  city: Bellevue
  state: Washington
  country: US
  email: jgalvin@identity.digital
- name: Michael Bauland
  org: Knipp Medien und Kommunikation GmbH
  city: Dortmund
  country: Germany
  email: michael.bauland@knipp.de

normative:
  RFC5730:

informative:
  RFC7940:
  EPDP:
    title: Phase 2 Initial Report of the EPDP on Internationalized Domain Names
    target: https://www.icann.org/en/public-comment/proceeding/phase-2-initial-report-of-the-epdp-on-internationalized-domain-names-11-04-2024
    date: 2024
    author:
      name: ICANN

--- abstract

This document defines an EPP extension allowing clients to learn about
and manipulate variant groups of domains, ie. groups of domains whose
names are equivalent in a registry-defined way and are tied to a
single registrant.

--- middle

# Introduction

EPP is defined in {{RFC5730}}. EPP commands were developed to operate on
a single object at a time. This document defines an EPP extension allowing
clients to learn about and manipulate variant groups of objects. 

Similar to EPP, the principal motivation for this extension was to provide a
standard Internet domain name registration extension for use between domain
name registrars and domain name registries.  This protocol provides a
means of interaction between a registrar's applications and registry
applications.  It is expected that this protocol will have additional
uses beyond domain name registration.

Spelling is not necessarily uniform. For example, an è and an e may be
regarded as equivalent in some languages, and as different in others.

Some registries plan to support this explicitly, with groups of
variant domains that can only be registered by the same registrant.
This document does not define a variant or a group of variants.
A registry policy MUST exist that specifies both that a registry supports
variant groups and that defines what domains are eligible to be a member
of a variant group. This policy MUST be agreed between a registry and a
registrar. The policy and the establishment of the agreement is outside
the scope of this specification.

A common policy expression among domain registries and registrars is to define
variants in terms of the script or language in use for an Internalized Domain
Name (IDN). IDN variants can arise when different characters or sequences of
characters in an IDN are considered equivalent in a particular language or script.
Standard Label Generation Rules (LGRs) are used to specify the IDN table that
establishes the variant relationships. This common policy is presumed and used
as an example in this specification.

With this extension, registering a domain creates a variant group and the first
domain in the group becomes the group's Primary Domain. The creation of the Primary
Domain MAY establish rules or guidelines regarding the domains that are eligible
to be a member of the group, e.g., an LGR, an IDN table, and a Primary Domain
taken together will define a variant group. Subsequent domains in
the same group can only be registered by the same registrar, which
asserts that it is acting on behalf of the same registrant. Each domain in
a variant group may be the target of any EPP command, with the following
restrictions.

* A &lt;transfer&gt; always acts on the entire group. This is required to
ensure that the variant group is always registered by the same registrant
and handled via the same registrar.

* The &lt;delete&gt; of the Primary Domain always deletes the entire group. This is
required to support the option where the Primary Domain establishes the rules or
guidelines for the creation of other domains in the group.

This extension is backwards compatible with registrars that do not support variant
groups. Specifically, this extension supports registries that do support variant
groups interacting with registrars that do not support variant groups. Registrars that do
not support variants that attempt to act on a member of a variant group inappropriately
will receive a compatible error response with which they can continue to function.
The compatible error response may not provide sufficient detail to fully understand
the rejection but will be sufficient to ensure continuation of normal operations.

The remainder of this document describes the specific details.

__TODO__: login exchange of variant-aware

__TODO__: reference to EPP Extensibility and Extension Analysis
https://docs.google.com/document/d/1WR00oB43XZCDqD0zvRvRajuWAq_9wQ3c0RrFKlGC3So/edit?tab=t.0

__TODO__: explain requirement for same registrant and the same entity principle

__TODO__: explain variant TLDs versus variant SLDs; define Primary TLD and Variant TLD

# Requirements Language

{::boilerplate bcp14-tagged}

# Terms

Same Entity Principle: A requirement that all domains within a variant group
either belong to the same entity (i.e., the same registrant via the same
registrar) or are withheld for that entity. No other entity is allowed to
activate any domain within the same variant group.

IDN Table: The combined information about what characters (code points)
are available for domain registration as well as the variant relationships
between those code points. IDN tables can be defined via RFC3743 or RFC4290
or LGRs (RFC7940). The latter one SHOULD be used as it also allows the
formal definition of context rules, which is lacking in the former ones.

Label Generation Rules (LGR): The preferred way of defining IDN tables.
Among others, they define the variant relationships as well as their
disposition values (blocked or allocatable). The formal definition of LGRs
can be fond in RFC7940.

Disposition Value: While a variant relationship is symmetric it has exactly
one of two disposition values which are not necessarily symmetric. A variant
can be "allocatable" (i.e., available for the same entity) or "blocked" 
(i.e., not available for anybody).

__TODO__: add Primary TLD and Variant TLD

Variant group: An implicit set of domains defined by the Label
Generation Rule (LGR) of the registry. The variant relationship is
symmetric and transitive. Hence, an arbitrary element of a variant set
defines the whole set. This set is not expressed explicitly in EPP,
because it can be impractically large. At the time of writing, a domain
is registered whose variant set would contain 10¹⁵ variants.

Primary domain: The chronologically first domain in a variant group.
While the variant relationship is symmetric, its disposition value is not.
It can either be blocked or allocatable. The primary domain name therefore
partitions the variant group into allocatable variants and blocked variants.
In case of variant TLDs, there can be a primary domain per TLD.

Variant domain: A domain in a variant group which is not a primary domain.

Allocated variant: A domain that has been created in the registry, and
which is tied to an existing primary domain.

Allocatable variant: A domain that has not been allocated but is allocatable
(according to the LGR), and which is conceptually tied to an existing
primary domain.

Activated variant: An allocated domain that is in the DNS.

__TODO__: Do we need to differentiate between allocated and activated? Does it
make any difference, whether a variant has DNS name servers or not?

Exempted domain: A preexisting domain that exists as a stand-alone domain,
but would be part of a variant group if it were allocated now. Exempted domains
may exist with any registrant at any registrar. The exemption stops
as soon as at most 1 allocated domain remains within a variant group.

Blocked domain: A domain that cannot be allocated due to its disposition
value in relation to the primary domain name.


# EPP Commands

## EPP &lt;check&gt; command

__TODO__: probably better not to have a command extension. For Registrars, it
would be difficult to determine the suitable primary domain. Therefore,
it is better to not ask them to send it, but rather return the appropriate
primary domain name in the response.

This extension defines a new command called the Variant Check Command
that defines an additional Primary Domain name element for the EPP
&lt;check&gt; command.

The command MAY contain an &lt;extension&gt; element, which MUST contain a
&lt;var:check&gt; element. The &lt;var:check&gt; element MUST contain one
&lt;var:primary&gt; element containing the intended primary domain.

~~~~~~~~~~~
C: <?xml version="1.0" encoding="utf-8" standalone="no"?>
C: <epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
C:   <command>
C:     <check>
C:       <domain:check
C:         xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
C:         <domain:name>examplev1.com</domain:name>
C:         <domain:name>examplev1.net</domain:name>
C:         <domain:name>examplev1.xyz</domain:name>
C:       </domain:check>
C:     </check>
C:     <extension>
C:       <var:check xmlns:var="urn:ietf:params:xml:ns:epp:variants-1.0">
C:         <var:primary>example.com<var:primary>
C:       </var:check>
C:     </extension>
C:     <clTRID>ABC-12345</clTRID>
C:   </command>
C: </epp>`
~~~~~~~~~~~

For the response, there is a distinction between variant-aware
clients (having supplied this extension durin the EPP &lt;login&gt;)
and clients that are agnostic of variants.

When the server receives a &lt;check&gt; command from a
variant-agnostic client and any domain within the checked domain's
variant group is allocated (or at least one exempted
domain within the variant group exists) the server MUST NOT include
an &lt;extension&gt; element. Instead, its response MUST
be available = "false". The optional reason MAY be
"Unavailable (except as variant)" to tell the registrar it
might be available as a variant.

When the server receives a &lt;check&gt; command from a variant-aware
client and the checked domain is part of a variant group with at least one
allocated variant (or exempted domain),
its response MUST contain an &lt;extension&gt; element, which MUST
contain a child &lt;var:chkData&gt; element. The &lt;fee:chkData&gt;
element MUST contain a &lt;var:cd&gt; element for each object referenced in
the client &lt;check&gt; command.

Each &lt;var:cd&gt; (check data) element MUST contain the following child
elements:

*  A &lt;var:objID&gt; element, which MUST match an element referenced in
the client &lt;check&gt; command.
*  An OPTIONAL &lt;var:primary&gt; element.
*  A &lt;var:status&gt; element, which explains in more detail the availability
status of the queried domain.

Example &lt;check&gt; response:

~~~~~~~~~~~
S: <?xml version="1.0" encoding="utf-8" standalone="no"?>  
S: <epp xmlns="urn:ietf:params:xml:ns:epp-1.0">  
S:   <response>  
S:     <result code="1000">  
S:       <msg>Command completed successfully</msg>  
S:     </result>  
S:     <resData>  
S:       <domain:chkData  
S:         xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">  
S:         <domain:cd>  
S:           <domain:name avail="1">examplev1.com</domain:name>  
S:         </domain:cd>  
S:         <domain:cd>  
S:           <domain:name avail="0">examplev1.net</domain:name>  
S:         </domain:cd>  
S:         <domain:cd>  
S:           <domain:name avail="0">examplev1.tel</domain:name>  
S:         </domain:cd>  
S:         <domain:cd>  
S:           <domain:name avail="0">examplev1.swiss</domain:name>  
S:         </domain:cd>  
S:       </domain:chkData>  
S:     </resData>  
S:     <extension>  
S:       <var:chkData  
S:           xmlns:var="urn:ietf:params:xml:ns:epp:variants-1.0">  
S:         <var:cd avail="1">  
S:           <var:objID>example.com</var:objID>  
S:           <var:primary>example.com</var:primary>  
S:           <var:status>AllocatableVariant</var:status>  
S:         </var:cd>  
S:         <var:cd avail="0">  
S:           <var:objID>example.net</var:objID>  
S:           <var:status>NotSameEntity</var:status>  
S:         </var:cd>  
S:         <var:cd avail="0">  
S:           <var:objID>example.tel</var:objID>  
S:           <var:status>Blocked</var:status>  
S:         </var:cd>  
S:         <var:cd avail="0">  
S:           <var:objID>example.swiss</var:objID>  
S:           <var:status>PendingTransfer</var:status>  
S:         </var:cd>  
S:       </var:chkData>  
S:     </extension>  
S:     <trID>  
S:       <clTRID>ABC-12345</clTRID>  
S:       <svTRID>54322-XYZ</svTRID>  
S:     </trID>  
S:   </response>  
S: </epp>  
~~~~~~~~~~~


The EPP &lt;check&gt; command may return five new results:


- The domain cannot be provisioned because it is a variant of a
Primary Domain, and the Primary Domain belongs to a different client  
=&gt; NotSameEntity

- The domain cannot be provisioned because its disposition value is blocked.  
=&gt; Blocked

- The domain cannot be provisioned because it is a variant of at
least one exempted domain.  
=&gt; Exempted

- The domain cannot be provisioned because it is a variant in a group
that is currently being transferred to a different registrar.  
=&gt; PendingTransfer

- Additional custom value that may be used for server peculiarities.  
=&gt; Custom


# EPP &lt;info&gt; command

Just like with the &lt;check&gt; there needs to be a distinction between
variant-aware and variant-agnostic clients. For variant-agnostic clients
there is no change to the standard behaviour. The response contains the
actual data of the domain, independent of the fact whether it is a variant
or not, in addition to the following:

* if the variant-agnostic registrar is inquiring about a non-allocated variant,
the response SHOULD be the same as the registrar inquiring about a reserved name.
If you don't have a policy, suggest a policy.

__TODO__: XML example of response?

For variant-aware clients, the EPP &lt;info&gt; command is not extended, 
but its response is extended
if the &lt;info&gt; command concerns a variant domain, i.e., at least two
domains within a variant group have been activated. The response then always
MUST include all primary domain names across all activated variant TLDs. Optionally
the response may include the list of all activated variants (across
all variant TLDs).

In case a Primary Domain name is queried in the &lt;info&gt; command, 
the list of activated variants within the same TLD MUST be returned.

In other words:
* If you ask about a primary domain name
  * you must return all primaries labels in all variant TLDs
  * you must return all activated variants in that TLD
  * you may return all activated variants in all variant TLDs

* if you ask about a variant
  * you must return the primary label for that variant
  * you must return all primary labels in all variant TLDs
  * you may return all activated variants in that TLD
  * you may return all activated variants in all variant TLDs

The main part of the response MUST contain the actual data of the queried 
domain name
(contacts, hosts, status values, etc.)

__TODO__: check whether EPP spec says anything about the alignment of check and info.

# EPP &lt;transfer&gt; command

If a variant-agnostic client initiates a transfer-in of a variant domain, i.e.,
at least two domains in a variant group have been activated, the transfer
request MUST be denied using 2305 "Object status prohibits operation".

__TODO__: xml example

If a variant-aware client initiates a transfer-in of a variant domain, i.e.,
at least two domains in a variant group have been activated, the transfer request
MUST include an extension specifying the Primary Domain for the indicated variant
domain, including if the Primary Domain is the variant domain indicated in the
transfer-in. If the extension is not present the transfer request MUST be denied
using 2003 "Required parameter missing".

The &lt;transfer&gt; is extended to include the complete group of all activated
variants formed by combining all variant groups from all variant TLDs.

__TODO__: It must be ensured that the poll message to the losing registrar
also contains the full list of domains that will be transferred together with
the primary domain.

__TODO__: xml example

# EPP &lt;create&gt; command

The EPP &lt;create&gt; command's task is to provision a new normal
domain. The task of converting an allocatable domain into an allocated
domain is instead performed using the &lt;update&gt; command.

If a variant-agnostic client initiates the creation of a domain that
is a member of the variant group of any variant TLD, independent of the status
of the domain in that group, the request MUST be rejected. The response SHOULD
be the same as if the domain to be created is reserved.
__ TODO__: make clear that registering the first domain within any variant 
group must not be rejected. The rejection only happens if at least one other
domain of that variant group already exists.

If a variant-agnostic client initiates the creation of a domain that does not
exist and is not a member of any variant group, then the following actions are
REQUIRED.

* If the create is otherwise permitted and the domain could be a Primary Domain,
the server MUST ensure that all eligible members of the variant group are
prevented from creation or allocation until such time as the domain
is expressly indicated by the client to be a Primary Domain.

* The server MUST act on the create and respond to the client as if the domain is
a new normal domain.

If a variant-aware client initiates the creation of a domain that is a member
of the variant group of any variant TLD, independent of the status of the domain
in that group, the command MUST be rejected with 2002 "Command use error".

If a variant-aware client initiates the creation of a domain that does not exist
and the client intends for the domain to be the Primary Domain of a variant group, the
create command MUST include an extension [[ specifying the Primary Domain of the
variant group ]] asserting that the client will ensure that only the same registrant
will request allocation of members of the variant group.

If a variant-aware client initiates the creation of a domain that does not exist and
the client does not include the extension [[ specifying the Primary Domain of the
variant group ]] then the following actions are REQUIRED.

* If the create is otherwise permitted and the domain could be a Primary Domain,
the server MUST ensure that all eligible members of the variant group are
prevented from creation or allocation until such time as the domain
is expressly indicated by the client to be a Primary Domain.

* The server MUST act on the create and respond to the client as if the domain is
a new normal domain.

The EPP &lt;create&gt; command may have five new errors, as described
in the &lt;check&gt; section above.

__TODO__: check alignment of the new error codes

__TODO__: do we want to allow variant activation also via the create command?
At least in the IDN EPDP we left this decision open to the registry policy.


# EPP &lt;update&gt; command

The EPP &lt;update&gt; command is extended to cover two new major
tasks:

* Activating a variant domain in an existing variant group

* Converting an Exempted Domain into a Primary Domain and optionally
converting other Exempted Domains that are eligible to be in the variant
group of the state Primary Domain to be activated domains of the
variant group.

This extended &lt;update&gt; is not valid for use by a variant-agnostic
registrar.  This rest of this section specifies behavior when variant-aware
registries and registrars are interacting.

When an EPP client wishes to provision a new domain in a variant
group, it uses the &lt;update&gt; command rather than the
&lt;create&gt; command. This informs the EPP server that the client
understands that the task is to provision a variant domain rather than
a new normal domain, and asserts that the two domains belong to the
same registrant.

Note that depending on registry policy, the variant domain may
e.g. share name servers with the Primary Domain. This implies that the
set of elements required/permitted for a variant domain may differ
from that of a Primary Domain or a normal domain.

__TODO__: update XML that shows the primary domain specified

__TODO__: specify creating a primary domain from an exempted domain without
other exempted domains

When an EPP client wishes to convert an exempted domain into a
Primary Domain and convert other exempted domains eligible to be members
of its variant group to be allocated domains, it issues an &lt;update&gt;
command including both the Primary Domain and separately the list
of exempted domains, which the EPP client thereby asserts belong to the
same registrant.

__TODO__: update XML that shows the list of variant domains specified

__TODO__: if variants are activated via the update command, shouldn't they
also be removed via the update command?

# EPP &lt;delete&gt; command

If a variant-agnostic client issues a &lt;delete&gt; command there is no change
from the standard functionality.

If a variant-aware client issues a &lt;delete&gt; command, the command is extended
to REQUIRE the client to include an extension indicating the Primary Domain of
the domain being deleted, which the client thereby asserts that both domains belong
to the same registrant.  If a Primary Domain is being deleted then the same
domain name MUST be specifed in the extension.

__TODO__: It should be left to server policy whether they want to delete the
whole variant set together with the primary domain or whether they require
all variants to be deleted individually first and only allow deletion of the
primary if no variant exists any more.

If a Primary Domain is to be deleted, the &lt;delete&gt; command is extended
with the following additional requirements:

* If the Primary Domain is part of variant group in a Variant TLD, then all variant
domains in the variant group of that Variant TLD MUST be deleted along with the
Primary Domain.  The response MUST include a list of all the allocated domains in the
variant group that were deleted.

* If the Primary Domain is part of a variant group in the Primary TLD, then all
variant domains all variant groups in all Variant TLDs MUST be deleted along with
the variant group of the Primary Domain in the Primary TLD.  The response MUST include a
list of all the allocated domains in all variant groups that were deleted.

__TODO__: xml example

# EPP renew command

The EPP renew command is not extended.

The server MAY reject renewals while a variant group is being
transferred.

# EPP &lt;transfer&gt; query command

The EPP &lt;transfer&gt; query command is not extended.

Note that because variant groups are transferred as a group, the
result of the a &lt;transfer&gt; query command is necessarily the same
for all domains in a group. Therefore, the result of &lt;transfer&gt;
query command for any domain in a variant group applies to all domains
in the group.

# Result codes

The following additional result codes are defined:

23x1: Change impossible because of a transfer in progress.

23x2: Change impossible because something is not a variant.

This error code is used when a change presupposes that two domains
belong to the same variant group, but the EPP server's implementation
disagrees.

23x3: Change impossible due to invalid primary domain

This error code is used when the primary domain specified in the
command is not registered, or is not registered via this registrar.

23x4: Change impossible due to unspecified primary domain

This error code is used when a command needs to specify a primary
domain, and does not.

23x5: Specified domain is exempted

This error code is used when a domain specifies a primary domain, and
the change is impossible because the specified domain is exempted
instead.

23x6: Specified domain is allocatable, but not by you.

This result code is used when a domain is a member of a variant set,
and the command did not refer to the primary domain.

# Acknowledgements

The design of this extension is almost completely based on work done
by and decisions made by the {{EPDP}} committee, which was reviewed by
a small technical design team chaired by James Galvin. Members of this
team included Dennis Tan, Rick Wilhelm, Edmon Chung, and Jennifer Chung.
This text (in RFC format) was initially written by Arnt
Gulbrandsen based on a conference presentation by James Galvin.

[YOUR NAME HERE] have reviewed it and provided helpful
comments or contributed in other ways.

# Security Considerations {#Security}

If two domains are different according to the DNS rules and identical
in the eyes of the intended audience, then the audience may be
confused. Confusion can always have security-related effects.

This extension expresses the relationships between variants clearly,
making it a little more difficult for a would-be impersonator to
register a variant of another registrant's existing domain.

# IANA Considerations {#IANA}

--- back

# Open issues

Open issue: Assign numbers to the error codes, properly.

Open issue: Not clear that there are any security considerations here
— the relationships between the domains may have some, but those exist
outside EPP, EPP merely describes them. In Italian, caffe and caffè
are variants of the same thing, it's not clear that linking them in a
protocol affects security in any way.

Open issue: Check how to insert a DS record in a variant domain.

Open issue: Can a unicode upgrade cause domains to become
exempted?  Yes, I think, and the terminology covers it, but as of
now, it's difficult for the EPP client to understand the situation.
Extending the &lt;info&gt; command would help here, perhaps.

Open issue: Creating a primary domain now consists of a two-step
process, &lt;create&gt; and then &lt;update&gt;. The first step turns
all variants into blocked domains, the second makes them
allocatable. It's not clear to me why the two-step process is
desirable, compared to a one-step process where a &lt;create&gt;
command creates a primary domain if the variant group contains other
domains than that one. That needs a couple of sentences of
explanation, or else a change.

Open issue: &lt;Delete&gt; now cascades and deletes many domains.
Should it instead turn any variant domains into exempted domains?

Open issue: Should the &lt;info&gt; of the primary domain also return
the list of allocated variants? Probably not — at the moment, this
extension has the client send a set that the server checks, and the
server may need to generate a set for checking, but the server does
not need to generate a list for returning.
