---
stand_alone: true
ipr: trust200902
cat: std
submissiontype: IETF
area: Applications
wg: EXTRA

docname: draft-ietf-regext-epp-variants-00

title: Domain variant support for EPP
abbrev: EPP Variants
lang: en
kw:
  - EPP
author:
- name: James Galvin
  org: Identity Digital
  city: Bellevue
  country: Washington
  email: jgalvin@identity.digital
- name: Michael Bauland
  org: Knipp Medien und Kommunikation GmbH
  city: Dortmund
  country: Germany
  email: michael.bauland@knipp.de

normative:
  RFC5730:

informative:
  RFC7940
  https://www.icann.org/en/public-comment/proceeding/phase-2-initial-report-of-the-epdp-on-internationalized-domain-names-11-04-2024

--- abstract

This document defines an EPP extension allowing clients to learn about
and manipulate variant groups of domains, ie. groups of domains whose
names are equivalent in a registry-defined way and are tied to a
single registrant.

--- middle

# Introduction

Spelling is not necessarily uniform. For example, an è and an e may be
regarded as equivalent in some languages, and as different in others.

Some registries plan to support this explicitly, with groups of
variant domains that can only be registered by the same registrant.

This document defines an EPP extension allowing clients to learn about
and manipulate variant groups. (EPP is defined in {{RFC5730}}.)

Registering a domain creates a variant group and the first domain in
the group becomes the group's primary domain. Subsequent domains in
the same group can only be registered by the same registrar, which
asserts that it is acting on behalf of the same registrant. Domains in
a variant group are transferred or deleted together with the primary
domain. The remainder of this document describes the specific details.

# Requirements Language

{::boilerplate bcp14-tagged}

# Terms

Label Generation Rules (LGR): A standard way of defining IDN tables.
Among others, they define the variant relationships as well as their
disposition values (blocked or allocatable).

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
but would be part of a variant group if it were allocated now. The same
entity principle does not apply to exempted domains. The exemption stops
as soon as at most 1 allocated domain remains within a variant group.

Blocked domain: A domain that cannot be allocated due to its disposition
value in relation to the primary domain name.


# EPP Commands

## EPP &lt;check&gt; command

__TODO__: probably better not to have a command extension. For Registrars, it
would be difficult to determine the suitable primary domain. Therefore,
it is better to not ask them to send it, but rather return the appropriate
primary domain name in the response.


`This extension defines a new command called the Variant Check Command
that defines an additional primary domain name element for the EPP
&lt;check&gt; command.

The command MAY contain an &lt;extension&gt; element, which MUST contain a
&lt;var:check&gt; element. The &lt;var:check&gt; element MUST contain one
&lt;var:primary&gt; element containing the intended primary domain.

C: &lt;?xml version="1.0" encoding="utf-8" standalone="no"?&gt;
C: &lt;epp xmlns="urn:ietf:params:xml:ns:epp-1.0"&gt;
C:   &lt;command&gt;
C:     &lt;check&gt;
C:       &lt;domain:check
C:         xmlns:domain="urn:ietf:params:xml:ns:domain-1.0"&gt;
C:         &lt;domain:name&gt;examplev1.com&lt;/domain:name&gt;
C:         &lt;domain:name&gt;examplev1.net&lt;/domain:name&gt;
C:         &lt;domain:name&gt;examplev1.xyz&lt;/domain:name&gt;
C:       &lt;/domain:check&gt;
C:     &lt;/check&gt;
C:     &lt;extension&gt;
C:       &lt;var:check xmlns:var="urn:ietf:params:xml:ns:epp:variants-1.0"&gt;
C:         &lt;var:primary&gt;example.com&lt;var:primary&gt;
C:       &lt;/var:check&gt;
C:     &lt;/extension&gt;
C:     &lt;clTRID&gt;ABC-12345&lt;/clTRID&gt;
C:   &lt;/command&gt;
C: &lt;/epp&gt;`

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

S: &lt;?xml version="1.0" encoding="utf-8" standalone="no"?&gt;
S: &lt;epp xmlns="urn:ietf:params:xml:ns:epp-1.0"&gt;
S:   &lt;response&gt;
S:     &lt;result code="1000"&gt;
S:       &lt;msg&gt;Command completed successfully&lt;/msg&gt;
S:     &lt;/result&gt;
S:     &lt;resData&gt;
S:       &lt;domain:chkData
S:         xmlns:domain="urn:ietf:params:xml:ns:domain-1.0"&gt;
S:         &lt;domain:cd&gt;
S:           &lt;domain:name avail="1"&gt;examplev1.com&lt;/domain:name&gt;
S:         &lt;/domain:cd&gt;
S:         &lt;domain:cd&gt;
S:           &lt;domain:name avail="0"&gt;examplev1.net&lt;/domain:name&gt;
S:         &lt;/domain:cd&gt;
S:         &lt;domain:cd&gt;
S:           &lt;domain:name avail="0"&gt;examplev1.tel&lt;/domain:name&gt;
S:         &lt;/domain:cd&gt;
S:         &lt;domain:cd&gt;
S:           &lt;domain:name avail="0"&gt;examplev1.swiss&lt;/domain:name&gt;
S:         &lt;/domain:cd&gt;
S:       &lt;/domain:chkData&gt;
S:     &lt;/resData&gt;
S:     &lt;extension&gt;
S:       &lt;var:chkData
S:           xmlns:var="urn:ietf:params:xml:ns:epp:variants-1.0"&gt;
S:         &lt;var:cd avail="1"&gt;
S:           &lt;var:objID&gt;example.com&lt;/var:objID&gt;
S:           &lt;var:primary&gt;example.com&lt;/var:primary&gt;
S:           &lt;var:status&gt;AllocatableVariant&lt;/var:status&gt;
S:         &lt;/var:cd&gt;
S:         &lt;var:cd avail="0"&gt;
S:           &lt;var:objID&gt;example.net&lt;/var:objID&gt;
S:           &lt;var:status&gt;NotSameEntity&lt;/var:status&gt;
S:         &lt;/var:cd&gt;
S:         &lt;var:cd avail="0"&gt;
S:           &lt;var:objID&gt;example.tel&lt;/var:objID&gt;
S:           &lt;var:status&gt;Blocked&lt;/var:status&gt;
S:         &lt;/var:cd&gt;
S:         &lt;var:cd avail="0"&gt;
S:           &lt;var:objID&gt;example.swiss&lt;/var:objID&gt;
S:           &lt;var:status&gt;PendingTransfer&lt;/var:status&gt;
S:         &lt;/var:cd&gt;
S:       &lt;/var:chkData&gt;
S:     &lt;/extension&gt;
S:     &lt;trID&gt;
S:       &lt;clTRID&gt;ABC-12345&lt;/clTRID&gt;
S:       &lt;svTRID&gt;54322-XYZ&lt;/svTRID&gt;
S:     &lt;/trID&gt;
S:   &lt;/response&gt;
S: &lt;/epp&gt;


The EPP &lt;check&gt; command may return five new results:


- The domain cannot be provisioned because it is a variant of a
primary domain, and the primary domain belongs to a different client
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

The EPP &lt;info&gt; command is not extended, but its response is extended
with the name of the primary domain if the &lt;info&gt; command concerns a
variant domain.

* If you ask about a primary domain name
* you must return all existing primaries
* you must return all activated variants in that TLD
* you may return all activated variants in all variant TLDs

* if you ask about a variant
* you must return the primary label for that variant
* you must return all primary labels in all variant TLDs
* you may return all activated variants in that TLDs
* you may return all activated variants in all variant TLDs

The main part of the response must contain the actual data of the queried domain name
(contacts, hosts, status values, etc.)

TODO: check whether EPP spec says anything about the alignment of check and info.

For registrars not supporting variants:
inquiring about a non-allocated variant should have the similar result as
inquiring about a reserved name.
If you don't have a policy, suggest a policy.


TODO XML example of response

# EPP &lt;transfer&gt; command

The EPP &lt;transfer&gt; command is modified as follows:

- A transfer of a primary domain also transfers all the variants
of that domain.

- A transfer of a variant domain returns en error.

__TODO__: There must be a safeguard for variant-agnostic clients to not
accidentally transfer-in a bundle of domains. Therefore, the transfer
command needs to have an extension, and a command must fail in case the
extension is not used, but there are variants.

# EPP &lt;create&gt; command

The EPP &lt;create&gt; command may have three new errors, as described
in the &lt;check&gt; section above.

The EPP &lt;create&gt; command's task is to provision a new normal
domain. The task of converting an allocatable domain into an allocated
domain is instead performed using the update command.

# EPP &lt;update&gt; command

The EPP &lt;update&gt; command is extended to cover two new major
tasks:

When an EPP client wishes to provision a new domain in a variant
group, it uses the &lt;update&gt; command rather than the
&lt;create&gt; command. This informs the EPP server that the client
understands that the task is to provision a variant domain rather than
a new normal domain, and asserts that the two domains belong to the
same registrant.

Note that depending on registry policy, the variant domain may
e.g. share name servers with the primary domain. This implies that the
set of elements required/permitted for a variant domain may differ
from that of a primary domain or a normal domain.

TODO update XML that shows the primary domain specified

When an EPP client wishes to turn an exempted domain into a
primary domain, it issues an &lt;update&gt; command including the list
of variant domains, which the EPP client thereby asserts belong to the
same registrant.

TODO update XML that shows the list of variant domains specified

# EPP &lt;delete&gt; command

The EPP &lt;delete&gt; command is extended with one new task: Deleting the
primary domain deletes all allocated variant groups along with the
primary domain.

Deleting a variant group other than the primary deletes just that
domain.

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
by and decisions made by the {{EPDP}} committee, chaired by James
Galvin. This text (in RFC format) was initially written by Arnt
Gulbrandsen based on a conference presentation by James Galvin.

Edmon Chung, [YOUR NAME HERE] have reviewed it, provided helpful
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
