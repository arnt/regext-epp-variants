---
stand_alone: true
ipr: trust200902
cat: std
submissiontype: IETF
area: "Applications and Real-Time"
wg: regext

docname: draft-galvin-regext-epp-variants-latest

title: Same Entity Set Support for EPP
abbrev: Same Entity Set
lang: en
kw:
  - EPP
author:
- name: James Galvin
  org: Identity Digital
  city: Bellevue, Washington
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
and manipulate a set of objects in a shared central repository that are
necessarily tied to the same entity (typically domain objects whose
names are equivalent in a registry-defined way and are tied to a
single registrant).  The extension supports multiple registries with a
shared definition of equivalence using a single service
provider with a shared central repository.

--- middle

# Introduction

EPP is defined in {{RFC5730}}. EPP commands were developed to operate
on a single object at a time. This document defines an EPP extension
allowing clients to learn about and manipulate a set of objects that
have a registry-defined characteristic that makes them equivalent
members of that set. As equivalent objects they MUST be tied to the
same entity, which this document calls the "Same Entity Principle".
When the Same Entity Principle (SEP) is active, the &lt;create&gt;,
&lt;delete&gt; and &lt;transfer&gt; transforms operate on "Same Entity
Sets".

Similar to EPP, the original motivation for this protocol was to
provide a standard Internet domain name registration protocol for use
between domain name registrars and domain name registries.  This
protocol provides a means of interaction between a registrar's
applications and registry applications.  It is expected that this
protocol will have additional uses beyond domain name registration.

With this extension, registering a domain creates a same entity set
and the first domain registered in the set becomes the set's Primary
Domain. (Domains are expected to be the most common kind of objects
involved, and word "domain" is generally used below.) If the Same
Entity Set's members share any options in a particular EPP server, as
defined by the registry, then these options are set using the
Primary. (Having the same name servers has been mentioned as an
example of this.)

Subsequent domains in the same set can only be registered in the
central repository by the same registrar. (In many cases, there will
be a contractual obligation to do so only on behalf of the same
registrant, but that is not within the area of EPP.) Each domain in a
same entity set may be the target of any EPP command, with the
following restrictions.

* A &lt;transfer&gt; of any domain in any same entite set always acts
on the entire set.  This is required to ensure that the same entity
set is always managed via the same registrar.

* The &lt;delete&gt; of a Primary Domain in any same entity set always
acts on the entire set. This is required to support the option where
the Primary Domain governs options as defined by the registry.

Note that a domain (or other object) is a member of a set by virtue of
a rule, not by virtue of having been created. Creating a Primary
Domain creates a set. The other members of the set thereby begin to
exist in a conceptual sense, and the set may be extremely large (a set
containing 10¹⁵ domains has been described in a realistic use case).
Therefore, this extension offers no way to enumerate the set's
members. The EPP client may need to list the allocated members of the
set, but the EPP server never lists the complete set. This leads to
the third restriction:

* A &lt;create&gt; of a later member of a Same Entity Set is not
possible; &lt;create&gt; creates a same entity set with at least one
allocated member.  Instead, &lt;update&gt; allocates an object in a
set, and makes it exist in the central repository.

This extension is backwards compatible with registrars that do not
support same entity sets. Registrars that attempt to act on a member
of a set inappropriately will receive a compatible error response with
which they can continue to function.  The compatible error response
may not provide sufficient detail to fully understand the rejection
but will be sufficient to ensure continuation of normal operations.

The remainder of this document describes the specific details.

__TODO__: discussion of reference to EPP Extensibility and Extension Analysis
https://docs.google.com/document/d/1WR00oB43XZCDqD0zvRvRajuWAq_9wQ3c0RrFKlGC3So/edit?tab=t.0



# Requirements Language

{::boilerplate bcp14-tagged}



# Terms

Allocated Member: A domain that has been created in the registry, and
which is related to an existing Primary Domain.

Allocatable Member: A domain that has not been allocated and exists
only conceptually in a Same Entity Set because it is related to the
aforementioned set's Primary Domain.

Activated Member: An Allocated Member domain that is in the DNS.

Blocked domain: A domain that cannot be allocated due to its status
value in relation to the Primary Domain name.

Exempted domain: A preexisting domain that exists as a stand-alone
domain prior to the introduction of support for this extension and
would be part of a set if it were allocated now. Exempted domains may
exist with any registrant at any registrar. The exemption ends in one
of two ways.

* When there is at most 1 allocated domain remaining, at which time
the EPP server MUST make the single allocated domain into a Primary
Domain of a Same Entity Set.

* When the registrar asserts knowledge of the Same Entity Set and, if
present, brings all domains in the Same Entity Set together.

Primary Domain: The chronologically first domain in a Same Entity Set.
While the member relationship in a Same Entity Set is symmetric, the
status value and other options of its members are not. For example, it
can either be blocked or allocatable. The Primary Domain name
therefore partitions the members of the Same Entity Set into
allocatable members and blocked members.  In the case of a Same Entity
Set of registries, there can be a Same Entity Set with a distinct
Primary Domain per Registry.

Same Entity Set: An implicit set of domains defined by a policy set by
a registry. The relationship between the members of the set is
symmetric. Hence, an arbitrary member of a same entity set defines the
whole set. The set is not expressed explicitly in EPP, because it can
be impractically large.

Same Entity Principle: A requirement that all domains within a set
either belong to the same EPP client or are withheld for that
client. No other client is allowed to allocate any domain within the
same set.

Status Value: While a related group relationship is symmetric, a
group member has exactly one of two status values which are not
necessarily symmetric. A member can be "allocatable" (i.e., available
for the same entity) or "blocked" (i.e., not available for anybody).



# Architectural Principles

There are three principles REQUIRED to be true at all times when this
extension is in use.  There MUST NOT be any exceptions at any time.

## Backwards Compatibility

Support for Same Entity Sets is optional and therefore it is REQUIRED
that a registry supporting Same Entity Sets MUST be backwards
compatible with a registrar that does not support Same Entity Sets.
Backwards compatibility is defined to mean that a registrar will
receive a response that is fail-safe including when the registrar may
not be able to fully understand the reason for the rejection.

A registry that does not support same entity sets will behave normally
when interacting with a registrar that supports related groups.

## Same-Entity Management

Domains defined to be eligible to be in a same entity set MUST be
managed by the same entity.  This has three requirements.

1. Registrars MUST ensure that domains in a same entity set are
managed by the same registrant.

2. Registries MUST ensure that domains in a same entity set are
managed by the same registrar.

3. Registries that are a member of a same entity set MUST be managed
by the same service provider.

## Related Groups as a Set

Most EPP commands may be executed independently on any member of the
same entity set.  However, commands that change the membership or
status of members in a same entity set, or change the Same-Entity
Management requirements, MUST operate on the members of the set as a
set.

As explained in detail in later sections, there are currently two
commands with explicit requirements as of the time of publication of
this document.


# Technical Principles

The following technical principles have guided the developed of this
extension and established operational requirements.

* The members of a same entity set are defined by registry policy and
that policy must be agreed by both the registry and the
registrar. The establishment of this policy and the method by which
the parties agree is outside the scope of this specification.

  * The first iteration of this work focused on IDN variants, which
    have the advantage that there is a relatively formal process for
    defining the eligible elements of a set. However, Latin
    characters with diacritic marks are not always considered variants
    of Latin characters without diacritic marks and there are
    circumstances when it is desirable for them to be considerated
    equivalent. As a result this extension presumes the existence of a
    set and sets outside its scope the actual definition of the
    members of the set.

* The registry policy MUST define the properties of a Same Entity Set,
which MUST include at least the following properties.

  * If the same entity set exists in a registry that itself is a
    member of a same entity set, then all same entity sets in any
    registry in the registry's same entity set MUST have the same
    members in all registries in the registry same entity set.

    This principle derives directly from the Same-Entity requirement.

    In the case of IDNs, the LGR tables may be different in each
    registry but the tables MUST be harmonized.

  * The first domain created in a same entity set is designated the
    Primary Domain.

    If the registry of the same entity set is itself a member of a
    same entity set, the Primary Domain in a same entity set MAY be
    different in each registry.


  * The Primary Domain has at least two REQUIRED functions.  First, it
    defines the members of the Same Entity Set.  Second, it defines the
    status values of the members of the Related Group.

* EPP today implicitly defines two status values for any domain:
  registered and available. This same entity set extension adds the
  following status values.

  The Allocated status means that the member of the set is active in
  the registry. It may or may not be delegated in the DNS.

  The Allocatable status means that the member of the set is
  available to be allocated by the same entity.

  The Blocked status means that the member of the set is not
  available to be allocated by anyone.

* The creation of a Primary Domain establishes the implicit existence
  of all members of the Same Entity Set. If the registry of the Same
  Entity Set is itself a member of a Same Entity Set, the Same Entity
  Set is implicitly created in all registries in the Same Entity Set
  of the registry.



# EPP Commands

In this section, the behavior of each EPP command when Related Groups
are supported is specified.


## EPP &lt;check&gt; command

The &lt;check&gt; command always acts on the target domain in the
command.  There is no change on the client side when using the
&lt;check&gt; command.

When the server receives a &lt;check&gt; command from a group-agnostic
client and the target domain is or could be a member of a related
group, if that related group has at least one Allocated or Exempted
member, the server's response:

* MUST NOT include the &lt;extension&gt; element.
* MUST indicate 'availabe = "false"'.
* MAY indicate a reason of "Unavailable (except as member of group)".

What the server receive a &lt;check&gt; command from a group-aware
client and the target domain is or could be a member of a related
group, if that related group has at least one Allocated or Exempted
member, the server's response MUST contain an &lt;extension&gt;
element with the following child elements:

* A &lt;var:primary&gt; element matching the Primary Domain for the
related group.
* A &lt;var:status&gt; element, which explains in more detail the availability
status of the queried domain.

The EPP &lt;check&gt; command may return six new results:

- AllocatableVariant: A variant of the domain is already active. Provisioning of this
domain must be to the same registrant via the same registrar.
- NotSameEntity: The domain cannot be provisioned because it is a variant of a
Primary Domain, and the Primary Domain belongs to a different client
- Blocked: The domain cannot be provisioned because its disposition value is blocked.
- Exempted: The domain cannot be provisioned because it is a variant of at
least one exempted domain.
- PendingTransfer: The domain cannot be provisioned because it is a variant in a group
that is currently being transferred to a different registrar.
- Custom: Additional custom value that may be used for server peculiarities.


## EPP &lt;info&gt; command

The &lt;info&gt; command always acts on the target domain in the
command.  There is no change on the client side when using the
&lt;info&gt; command.

The main part of the response MUST contain the actual data of the target
domain name (contacts, hosts, status values, etc.).

When the server receives an &lt;info&gt; command from a group-agnostic
client the response MUST contain the actual data of the domain,
independent of whether it is a member of a related group. In addition,
if the group-agnostic registrar is inquiring about a domain with a
status of Allocatable, the response SHOULD be the same as if the
client were inquiring about a reserved name.

When the server receives an &lt;info&gt; command from a group-aware
client and the target domain is or could be a member of a related
group, if that related group has at least one Allocated or Exempted
member, the server's response MUST contain an &lt;extension&gt;
element with the following child elements:

* A &lt;var:primary&gt; element matching the Primary Domain for the
related group of the target domain, which MAY match the target domain.
* A list of all the Allocated and Exempted members of the related group.

If the registry of the target domain is itself a member of a related
group and the target domain is or could be a member of a related group
in any registry in that registry's related group, if any one of those
target domain related groups has at least one Allocated or Exempted
member, the server's response MUST contain an &lt;extension&gt;
element with the following child elements:

* A &lt;var:primary&gt; element matching the Primary Domain for the
related group of the target domain, which MAY match the target domain.

* A list of all the related groups of the target domain with Allocated
or Exempted members such that each related group list has its
Primary Domain listed first.



## EPP &lt;transfer&gt; command

The &lt;transfer&gt; command always acts on the target domain in the
command.  The use of the &lt;transfer&gt; command is extended if both
the server and the client support related groups.

When the server receives a &lt;transfer&gt; command from a
group-agnostic client and the target domain is or could be a member of
a related group, if that related group has at least one Allocated or
Exempted member the transfer request MUST be denied using 2305 "Object
status prohibits operation".

When the server receives a &lt;transfer&gt; command from a group-aware
client and the target domain is or could be a member of a related
group, the request must include an &lt;extension&gt; element with a
&lt;var:primary&gt; element matching the Primary Domain, including if
the Primary Domain is the target domain.  If the extension is not
present the transfer request MUST be denied using '2003 "Required
parameter missing"'. Note that the &lt;check&gt; or &lt;info&gt;
command MAY be used to identify the Primary Domain.

A valid transfer request MUST apply to all members of a related group.
If the registry of the target domain is itself a member of a related
group, then the transfer request MUST apply to all related groups in
all registries of the registry's related group.

The server's response to the transfer request MUST contain an
&lt;extension&gt; element with the following child elements;

* A &lt;var:primary&gt; element matching the Primary Domain for the
related group of the target domain, which MAY match the target domain.
* A list of all the Allocated and Exempted members of the related group.

If the registry of the target domain is itself a member of a related
group and the target domain is or could be a member of a related group
in any registry in that registry's related group, if any one of those
target domain related groups has at least one Allocated or Exempted
member, the server's response MUST contain an &lt;extension&gt;
element with the following child elements:

* A &lt;var:primary&gt; element matching the Primary Domain for the
related group of the target domain, which MAY match the target domain.

* A list of all the related groups of the target domain with Allocated
or Exempted members such that each related group list has its
Primary Domain listed first.

__TODO__: It must be ensured that the poll message to the losing registrar
also contains the full list of domains that will be transferred together with
the primary domain.



## EPP &lt;create&gt; command

The EPP &lt;create&gt; command's standard task is to provision a new
domain. When related groups are supported, the &lt;create&gt; command
MUST be used to create the Primary Domain and MUST NOT be used to
provision any other member of the Primary Domain's related group. The
task of converting an allocatable domain into an allocated domain MUST
be performed using the &lt;update&gt; command.

When the server receives a &lt;create&gt; command from a
group-agnostic client and the target domain is or could be a member of
a related group, one of the following actions MUST be completed as
appropriate.

* If any member of the related group is currently Allocated or
Exempted, the command MUST be rejected and the response should be the
same as if the domain to be created is reserved.

* If there are no members of the related group either Allocated or
Exempted, the &lt;create&gt; MUST proceed according to the standard
with the server implicitly reserving all other members of the related
group such that they MUST NOT be allocated until such time as the
client is group-aware and the client MUST indicate that the target
domain is to be extended to be a Primary Domain as described in the
&lt;update&gt; command.

When the server receives a &lt;create&gt; command from a group-aware
client and the target domain is or could be a member of a related
group, the command MUST include an &lt;extension&gt; element with the
&lt;var:primary&gt; child element that must match the target
domain. If not, the command MUST be rejected using '2003 "Required
parameter missing"'.

Upon receiving a &lt;create&gt; command from a group-aware client with
a valid &lt;extension&gt; element, one of the following actions MUST
be completed as appropriate.

* If the target domain does not exist and any other member of the
related group is Allocated or Exempted, the command MUST be rejected
and indicate that it is an inappropriate use of the command.

* If the target domain does not exist and no other member of the
related group is Allocated or Exempted, the &lt;create&gt; command
MUST proceed according to the standard with the server implicitly
noting to itself the existence of all other members of the related
group and setting their status value as prescribed by registry
policy. The response MUST include the &lt;extension&gt; element with
the &lt;var:primary&gt; child element indicating the Primary Domain,
which must match the target domain.

* If the target domain does exist, the &lt;create&gt; command MUST be
rejected according to the standard. The response MUST include the
&lt;extension&gt; element with the &lt;var:primary&gt; child element
indicating the Primary Domain, which must match the target domain.

The EPP &lt;create&gt; command may have five new errors, as described
in the &lt;check&gt; section above.

__TODO__: check alignment of the new error codes



## EPP &lt;update&gt; command

The EPP &lt;update>gt; command provides a transform operation that allows a
client to change the state of a related domain object. It is extended
to cover three new tasks:

* Activating an allocatable related domain in an existing related group.
* Deactivating an activated related domain name in an existing related group.
* Converting an Allocated or Exempted Domain into a Primary Domain and
optionally converting other Exempted Domains that are eligible to be
in the related group of the stated Primary Domain to be activated
domains of the related group.

This extended &lt;update&gt; command is not valid for use by a
group-agnostic client. Any use by a group-agnostic client MUST be
rejected and indicate it is an inappropriate use of the command.

A group-agnostic client MUST only use the standard-defined
&lt;update&gt; command and the server MUST only respond as defined by
the standard.

The rest of this section specifies behavior when group-aware servers
and group-aware clients are interacting and describes the three new
tasks.

When the target domain of the update command is any member of the
related group, including the Primary Domain of the related group, the
client MUST include an &lt;extension&gt; element that MUST include at
least the &lt;var:primary&gt; child element indicating the Primary
Domain of the corresponding related group. The extension MAY include
additional elements as indicated below to provision a new task. If the
extension is not present the command MUST be rejected and indicate
that a required parameter is missing.

If the Primary Domain and the target domain match, all other elements
in the extension MUST be ignored and the update command MUST be
processed as a standard defined update command acting on the Primary
Domain.

The rest of this section specifies behavior when the target domain and
the Primary Domain indicated in the extension do not match.

If the &lt;var:status&gt; child element is present in the extension,
one of the following actions MUST be completed as appropriate.

* In order to Activate an Allocatable domain, the target domain MUST
have a status of Allocatable and the extension MUST include the
&lt;var:status&gt; child element with a value of "allocated". The
server MUST update the status of the target domain and the response
MUST include the extension element with both the Primary Domain
indicated and the revised status indicated.

* In order to deactivate an Allocated domain, the target domain MUST
have a status of Allocated and the extension MUST include the
&lt;var:status&gt; child element with a value of "allocatable". The
server MUST update the status of the target domain and the response
MUST include the extension element with both the Primary Domain
indicated and the revised status indicated.

* In all other cases, if the status element is present the command
MUST be rejected and indicate an invalid parameter is present.

If the &lt;var:status&gt; child element is not present in the
extension, then all elements other than the Primary Domain indication
MUST be ignored and the update command MUST be processed as a standard
defined update command acting on the target domain.

Note that depending on registry policy, the related domain may share
attributes with the Primary Domain, e.g., nameservers.  A registry
policy MAY specify rules or guidelines for the set of elements
required or permitted for a related domain according to the Primary
Domain.

The EPP domain mapping from RFC3915 describes the elements that
have to be specified within an &lt;update>gt; command.  The requirement to
provide at least one &lt;domain:add>gt;, &lt;domain:rem>gt;, or &lt;domain:chg>gt;
element is updated by this extension such that at least one empty
&lt;domain:add>gt;, &lt;domain:rem>gt;, or &lt;domain:chg>gt; element MUST be present
if this extension is specified within an &lt;update>gt; command.  This
requirement is updated to disallow the possibility of modifying a
domain object as part of the deactivation.

If a client wishes to convert an Exempted domain into the Primary
Domain of a related group, the update command from the client MUST be
provided as follows.

* The target domain of the update command and the Primary Domain in
the extension MUST match.
* The target domain MUST have the status of Exempted.
* If there exists multiple Exempted domains that would ordinarily be
members of the related group, they MUST all have the same Registrar of
Record and it MUST match the update requesting registrar, and the extension
MUST include a list of all Exempted domains, including the
Primary Domain, that MUST match the list maintained by the registry.

If the update command is valid as indicated above, the server MUST
change the status of the indicated domains from Exempted to Allocated,
and MUST indicate the Primary Domain. The response MUST include an
extension indicating the Primary Domain and the list of domains whose
status changed from Exempte to Allocated.

If a previously group-agnostic client becomes group-aware and wishes
to convert a registered domain to be a Primary Domain of related
group, the update command from the client MUST be provided as follows.

* The target domain of the update command and the Primary Domain in
the extension MUST match.
* The target domain MUST have the status of registered and MUST have
the same Registrar of Record as the update requesting registrar.

If the update command is valid as indicated above, the server MUST
change the status of the indicated domains to Allocated, and MUST
indicate it as the Primary Domain. The response MUST include an
extension indicating the Primary Domain.



## EPP &lt;delete&gt; command

The &lt;delete&gt; command is extended to REQUIRE the deletion of all
members of a related group if the Primary Domain is deleted.

This extended &lt;delete&gt; command is not valid for use by a
group-agnostic client. Any use by a group-agnostic client MUST be
rejected and indicate it is an inappropriate use of the command.

A group-agnostic client MUST only use the standard-defined
&lt;update&gt; command and the server MUST only respond as defined by
the standard.

The rest of this section specifies behavior when group-aware servers
and group-aware clients are interacting.

A group-aware client MUST NOT use the &lt;delete&gt; command to delete
any member of a related group except the Primary Domain. Any other use
MUST be rejected and indicate it is an inappropriate use of the
command. Note that &lt;update&gt; command is used for this purpose.

When the server receives a &lt;delete&gt; command from a group-aware
client, the command MUST include the &lt;extension&gt; element which
MUST include the &lt;var:primary&gt; child element which MUST match
the target domain name.

The delete command is extended such that all Allocated members of the
related group defined by the Primary Domain MUST all be deleted at
once. If it is not possible for any member of the related group to be
deleted for any reason, the delete command MUST fail leaving all
members of the related group intact.

If the delete command is successful, the response MUST include the
extension element which MUST include both an indication of the Primary
Domain and the list of all members of the related group that were
deleted, including the Primary Domain.



## EPP renew command

The EPP renew command is not extended.

The server MAY reject renewals while a related group is being
transferred.


## EPP &lt;transfer&gt; query command

The EPP &lt;transfer&gt; query command is not extended.

Note that because related groups are transferred as a group, the
result of the a &lt;transfer&gt; query command is necessarily the same
for all domains in a group. Therefore, the result of a
&lt;transfer&gt; query command for any domain in a related group
applies to all domains in the group.



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

YOU YES YOU (<- insert name) have reviewed it and provided helpful
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

Open issue: &lt;Delete&gt; now cascades and deletes many domains.
Should it instead turn any variant domains into exempted domains?
