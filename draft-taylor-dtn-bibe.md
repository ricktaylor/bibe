---

title: Bundle-in-Bundle Encapsulation
abbrev: BIBE
category: std

docname: draft-taylor-dtn-bibe-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
# area: INT
# workgroup: Delay/Disruption Tolerant Networking

keyword:

- DTN
- BPv7
- Bundle Protocol
- Encapsulation

author:

- fullname: Rick Taylor
  role: editor
  organization: Aalyria Technologies
  email: rtaylor@aalyria.com

# EDNOTE: This document replaces the expired draft-ietf-dtn-bibect-05 under a

# new name; the datatracker "Replaces: draft-ietf-dtn-bibect" relationship

# should be recorded on submission. The -05 authors are listed as Contributors

# below; individuals to confirm they are content with Contributor (rather than

# co-author) listing, and affiliations/addresses to be re-verified before

# submission

contributor:

- fullname: Scott Burleigh
  organization: IPNGROUP
  email: sburleig.sb@gmail.com
  contribution: Author of the successive BIBE Internet-Drafts, from draft-irtf-burleigh-bibe (2013) through draft-ietf-dtn-bibect, from which this document is directly derived; the BIBE architecture is his.

- fullname: Alberto Montilla
  organization: Spatiam Corporation
  email: a.montilla@spatiam.com
  contribution: Co-author of draft-ietf-dtn-bibect.

- fullname: Joshua Deaton
  organization: SAIC
  email: joshua.e.deaton@nasa.gov
  contribution: Co-author of draft-ietf-dtn-bibect, and contributor to the CCSDS Orange Book segmentation work on which the segmentation mechanism of this document builds.

- fullname: Carlo Caini
  organization: University of Bologna
  email: carlo.caini@unibo.it
  contribution: Co-author of draft-ietf-dtn-bibect.

venue:
  group: Delay/Disruption Tolerant Networking
  type: Working Group
  mail: dtn@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/dtn/
  github: ricktaylor/bibe
  latest: https://ricktaylor.github.io/bibe/draft-taylor-dtn-bibe.html

normative:

informative:
  BTPU: I-D.ietf-dtn-btpu
  GRE: RFC2784
  GREKEY: RFC2890
  IPIP: RFC2003
  TUNMTU: RFC4459
  TUNLIMIT: RFC2473
  TUNSEC: RFC6169

--- abstract

This document describes Bundle-in-Bundle Encapsulation (BIBE), a Delay-Tolerant Networking (DTN) Bundle Protocol (BP) tunneling mechanism by which a bundle is carried as the payload of one or more encapsulating bundles, allowing security measures, routing policy, and protocol version translation to be applied to the encapsulating bundle without modification of the encapsulated bundle. The protocol includes an optional segmentation mechanism, allowing a large encapsulated bundle to be carried in multiple encapsulating bundles.

--- middle

# Introduction

Bundle Protocol version 7 (BPv7) {{!RFC9171}} defines a layered architecture in which a Bundle Protocol Agent (BPA) relies on convergence-layer adapters (CLAs) to transfer bundles between nodes. This document defines Bundle-in-Bundle Encapsulation (BIBE), in which the transfer between two nodes is performed by a BP network itself: an outbound bundle (the "encapsulated bundle") is carried as the payload of one or more bundles (the "encapsulating bundles") which traverse the BP network between the encapsulating node and the decapsulating node in the normal manner. [^editorial]

[^editorial]: This document is an editorial consolidation of prior work: the architecture and concepts originate in draft-ietf-dtn-bibect and the CCSDS Bundle-in-Bundle Encapsulation effort, and the design contributions of others are recorded in the Contributors and Acknowledgments sections. The editor claims no design novelty; the contribution of this document is to restate that body of work as a single self-contained specification.

BIBE thus occupies the architectural position of a convergence layer — the "link layer" beneath it is itself a BP network — but this document deliberately does not require that it be implemented as a convergence-layer adapter; it specifies observable behavior only, in terms of the encapsulation function and decapsulation element defined in {{model}}.

Conformance to this specification is OPTIONAL for BP nodes. A conforming node may implement the encapsulation function, the decapsulation element, or both; a unidirectional tunnel requires only one at each end.

## Use Cases {#use-cases}

BIBE has broad utility; motivating use cases include:

- Security encapsulation. BPSec {{?RFC9172}} requires that the primary block of a bundle remain in plaintext so that it can be routed. Encapsulating a bundle allows a Bundle Confidentiality Block on the encapsulating bundle's payload to encrypt the entire encapsulated bundle, including its source and destination, providing a defense against traffic analysis that BPSec alone cannot offer.
- Cross-domain and service-provider encapsulation. A bundle arriving at the edge of a network domain can be encapsulated so that domain-specific policies (forwarding, quality of service, security) are applied to the encapsulating bundle, leaving the encapsulated bundle unmodified. The encapsulating bundle egresses the domain at a designated decapsulating node.
- Version encapsulation. A Bundle Protocol version 6 {{?RFC5050}} bundle can be encapsulated for carriage across a BPv7 network, or vice versa.
- Segmentation. BPv7 bundle fragmentation ({{Section 5.8 of RFC9171}}) is incompatible with some BPSec deployments and is prohibited for bundles bearing certain security blocks. The segmentation mechanism defined in this document ({{segmentation}}) provides an alternative: a large bundle is carried opaquely as chunks in multiple smaller encapsulating bundles, which is of particular utility when a downstream convergence-layer adapter cannot carry large bundles.
- Traffic differentiation. Decapsulation endpoints are ordinary endpoints established by policy ({{model}}), so a decapsulating node may establish several — per tunnel, per peer, or per traffic class — at no protocol cost. The destination EID of each encapsulating bundle then acts as a flow identifier visible along the encapsulating path: nodes on the tunnel path can apply forwarding, queueing, and quality-of-service policy to tunnel traffic by destination EID alone, without inspecting the payload — including when the payload is encrypted. The same visibility necessarily reveals that class structure to on-path observers ({{security-considerations}}).

As in {{BTPU}}, the encapsulated bundle is treated throughout as an opaque sequence of octets: no field of this protocol identifies its format, and nothing in the procedures of this document depends upon it. Bundles of different Bundle Protocol versions can therefore be carried, and multiplexed to the same decapsulation endpoint, without an explicit discriminator: a receiver can distinguish the formats by simple examination of the initial octets of the decapsulated data (a BPv6 bundle begins with the octet 0x06, the BPv6 version number, whereas a BPv7 bundle begins with the initial octet of a CBOR array). Indeed, any binary data can be carried; the disposition of decapsulated data that is not a bundle is beyond the scope of this document and is a matter for the receiving implementation.

Encapsulating bundles are themselves bundles and may in turn be encapsulated, enabling nested encapsulation to arbitrary depth as required by policy.

## Similarity to IP Tunneling

BIBE shares a pattern with generic IP tunneling, and this document deliberately follows the same architectural conventions where the analogy holds:

- Like GRE {{GRE}} and IP-in-IP {{IPIP}}, BIBE is a stateless encapsulation: there is no tunnel session, no negotiation, and no convergence-layer acknowledgement. The tunnel is defined entirely by configuration at its two endpoints.
- Encapsulation is a tunnel ingress operation: routing or forwarding policy directs an encapsulated bundle "into" the tunnel exactly as an IP route directs a packet to a tunnel interface. An implementation that realizes the encapsulation function as a virtual interface (a convergence-layer adapter) selected by a forwarding table entry mirrors a multipoint GRE interface; an implementation that applies it as egress policy mirrors policy-based tunnel selection. Either way, a single encapsulation function may serve multiple decapsulation endpoints — and, conversely, several decapsulation endpoints at one node give the encapsulating node a per-class tunnel selector, in the manner of the GRE Key field {{GREKEY}} ({{use-cases}}).
- Applying a Bundle Confidentiality Block to the encapsulating bundle's payload is the analog of IPsec tunnel mode: the entire inner datagram, including its addressing, is protected, whereas BPSec applied directly to a bundle (like transport mode) must leave the primary block in plaintext for routing.
- The segmentation mechanism of {{segmentation}} is the analog of "outer fragmentation" in IP tunnels ({{TUNMTU}}): the tunneled payload is divided by and reassembled at the tunnel endpoints, transparently to the encapsulated bundle, rather than fragmenting the encapsulated bundle itself ("inner fragmentation", i.e. BPv7 bundle fragmentation) which alters the tunneled data unit and interacts poorly with BPSec.

IP tunneling experience also carries over its known hazards: recursive encapsulation loops (see {{deployment}}) and the security concerns cataloged in {{TUNSEC}} (see {{security-considerations}}).

# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Terminology

Encapsulated bundle:
: A bundle, of any Bundle Protocol version, forwarded by encapsulation and carried opaquely as (part of) the payload of one or more encapsulating bundles.

Encapsulating bundle:
: A BPv7 bundle whose payload encapsulates all or part of another bundle, formatted as described in {{payload-format}}. The payload takes one of two forms: a "whole-bundle payload" carrying an entire encapsulated bundle, or a "segment payload" carrying part of one.

Encapsulation function:
: The function of an encapsulating node that generates encapsulation payloads from a bundle being forwarded and sources the encapsulating bundles that carry them. See {{model}}.

Decapsulation element:
: The element of the application agent of a decapsulating node that takes delivery of encapsulation payloads, performs reassembly when required, and presents encapsulated bundles to the BPA. See {{model}}.

Decapsulation endpoint:
: An endpoint in which a decapsulation element is registered; the destination endpoint of encapsulating bundles.

Transfer:
: The context in which the segments of a single encapsulated bundle are transmitted, identified by a Transfer ID.

# Functional Model {#model}

BIBE is specified as observable behavior, not as a component architecture, in terms of two entities:

- An "encapsulation function", invoked by the BPA of the encapsulating node in the course of forwarding a bundle: it generates one or more payloads, each encapsulating all or part of the bundle being forwarded, and, acting in the capacity of a bundle source, sources the encapsulating bundles that carry them ({{transmission}}). Each encapsulating bundle is a new bundle, originated at the encapsulating node; see {{encap-bundle}}.
- A "decapsulation element", registered in one or more endpoints of the decapsulating node: it takes delivery of the payloads of encapsulating bundles, performs reassembly when required, and, acting in the capacity of a convergence-layer adapter, presents encapsulated bundles to the BPA for forwarding and/or delivery in the standard manner ({{reception}}).

{{fig-model}} illustrates the asymmetry of the model: the encapsulation function operates on the forwarding path of the encapsulating node, while the decapsulation element is an element of the application agent of the decapsulating node; the encapsulated bundle experiences the entire tunnel as a single hop.

~~~aasvg
      Encapsulating Node                    Decapsulating Node

 bundle B ...(to bundle B, the tunnel is one hop)... bundle B
    |                                                    ^
    |                                                    |
 (forwarding path)                          (application agent)
    v                                                    |
 +--------------+                                +--------------+
 | encapsulation|                                | decapsulation|
 |   function   |                                |    element   |
 +--------------+                                +--------------+
    |                                                    ^
    | sources bundle E                     payload of E  |
    | (payload carries B,                  delivered to  |
    |  destined for the                    the decap     |
    |  decap endpoint)                     endpoint      |
    v                                                    |
   BPA --CLA--> node --CLA--> ... --CLA--> BPA ----------+
            ordinary BPv7 forwarding of bundle E
~~~
{: #fig-model title="Functional model: encapsulation of bundle B in bundle E"}

How a BPA selects the encapsulation function is an implementation matter: it might be realized as a convergence-layer adapter selected by a forwarding table entry, applied as a filter or transform on the egress forwarding path, or integrated in some other way. Nothing in this document depends on the choice.

The decapsulation element, by contrast, cannot be realized solely as a convergence-layer adapter: it takes delivery of the payloads of bundles addressed to it, and it therefore MUST be registered in one or more endpoints — in the terms of {{Section 3.2 of RFC9171}}, it acts in the capacity of an element of the node's application agent. When it presents an encapsulated bundle to the BPA, however, it presents a received bundle for the reception processing of {{Section 5.6 of RFC9171}} ({{reception}}), and in doing so acts in the capacity of a convergence-layer adapter. Nothing in this document prevents a single component from acting in both capacities — indeed, taking delivery of encapsulation payloads in the capacity of an application-agent element and presenting encapsulated bundles in the capacity of a convergence-layer adapter is the natural realization.

The choice of endpoints in which a decapsulation element is registered is a matter of administrative policy at the decapsulating node. A BIBE tunnel is a deliberate, bilaterally configured arrangement — an encapsulating node must in any case hold per-peer configuration ({{deployment}}), of which the decapsulation endpoint ID is simply one element — so there is no unsolicited-contact scenario for which a "well-known service" convention would be required, and the absence of one denies adversaries a predictable target for unsolicited traffic ({{security-considerations}}). A decapsulation endpoint SHOULD be an endpoint formed in a URI scheme whose procedures define which member or members of the endpoint take delivery of a bundle destined for it. A singleton endpoint satisfies this trivially; so would an endpoint in a scheme with defined multicast delivery semantics — delivery to every member — which would realize multicast BIBE: encapsulated forwarding to every member of the decapsulation endpoint at once. What is to be avoided is an endpoint with more than one member for which the choice of delivering member is left unspecified, as the Bundle Protocol leaves it for multi-member endpoints in the dtn scheme; the consequences of that indeterminacy for BIBE — unpredictable duplication of whole-bundle payloads, and segmented transfers that can never be reassembled — are set out in {{segmentation}}.

Note that BIBE encapsulation is a forwarding operation, not a delivery operation: policy (including BPSec) applicable to the forwarding path of the encapsulating bundle applies to it in the normal way. The payloads generated by the encapsulation function are plaintext; any confidentiality protection is applied by the BPA in the course of transmitting the encapsulating bundle, exactly as for any other bundle payload.

# Procedures

## Transmission {#transmission}

When the BPA of the encapsulating node determines — by forwarding table entry, forwarding policy, or other configuration — that a bundle is to be forwarded by encapsulation toward a specified decapsulation endpoint, the bundle is presented to the encapsulation function.

The bundle so presented is the bundle as the encapsulating node forwards it. The forwarding-time processing that precedes transmission over any convergence-layer adapter — in particular the Previous Node block replacement of {{Section 4.4.1 of RFC9171}}, subject to the encapsulating node's policy — is performed by the BPA in the normal manner, exactly as when forwarding via any other convergence-layer protocol; it is not a function of the encapsulation function. Because this document does not require that the encapsulation function be realized as a convergence-layer adapter ({{model}}), however, the capture point must be pinned explicitly: the octets captured for encapsulation SHOULD be those of the bundle after that forwarding-time processing, so that, for example, a Previous Node block frozen into the encapsulated bundle identifies the encapsulating node. The consequences for the blocks of the encapsulated bundle are set out in {{encap-bundle}}.

The encapsulation function then generates the payload or payloads:

- If the length of the bundle does not exceed the segmentation threshold for that endpoint ({{segmentation}}), the encapsulation function SHALL generate a single whole-bundle payload encapsulating the entire bundle, and SHALL request transmission of an encapsulating bundle, destined for that endpoint, carrying that payload. A bundle that can be carried whole SHOULD NOT be segmented.
- Otherwise the bundle SHALL be segmented as described in {{segmentation}}. If the bundle cannot be segmented — the encapsulation function does not support segmentation, or the decapsulation element is not known to support reassembly ({{reception}}) — then forwarding by encapsulation has failed, and the forwarding contraindication procedures of {{Section 5.4 of RFC9171}} apply at the encapsulating node as for a failure of any convergence layer.

The fields and blocks of each encapsulating bundle are populated as described in {{encap-bundle}}.

Upon requesting transmission of the encapsulating bundle(s), forwarding of the encapsulated bundle at the encapsulating node is complete; no further state regarding it need be retained. BIBE provides no acknowledgement of its own, so "complete" means successful hand-off to the underlying network, as with any unreliable convergence layer ({{deployment}}).

## Segmentation {#segmentation}

For each decapsulation endpoint to which encapsulating bundles will be sent, a maximum segment size — the endpoint's "segmentation threshold" — SHALL be established by a-priori configuration ({{deployment}}). The segmentation threshold accommodates, for example, size limits of convergence-layer adapters on the path of the encapsulating bundles. Note that the threshold bounds encapsulated-bundle octets per encapsulating bundle: an implementation sizing encapsulating bundles for a downstream limit must also account for the encapsulating bundle's own overhead (primary block, extension blocks, payload CBOR framing, and any BPSec blocks).

The segmentation mechanism is aligned with the transfer/segment model of {{BTPU}}, but requires none of BTPU's padding, message framing, or transfer-window machinery: BPv7 itself provides the "framing layer", guaranteeing that the payload of each encapsulating bundle is delivered whole or not at all, so only segment identification and reassembly rules are needed.

When a bundle is to be segmented:

- The encapsulation function SHALL assign the transfer a Transfer ID as described in {{transfer-ids}}.
- The encapsulation function SHALL partition the octets of the bundle into two or more segments, each no larger than the segmentation threshold, covering every octet of the bundle exactly once, in order, without alteration.
- For each segment the encapsulation function SHALL generate a segment payload with the common transfer-id, the total length of the bundle, the segment's offset, and the segment's octets, and SHALL request transmission of an encapsulating bundle, destined for the indicated endpoint, carrying that payload.

Requesting transmission of the first encapsulating bundle of a transfer commits the encapsulation function to the remainder: this protocol provides no abort or cancellation signal, so a transfer abandoned partway is indistinguishable, at the decapsulating node, from loss of the remaining encapsulating bundles in transit ({{deployment}}). An abandoned transfer occupies reassembly state at the decapsulation element until the transfer's expiry ({{resources}}), and its Transfer ID remains unavailable for reuse until that same time ({{transfer-ids}}).

Segmentation requires that any decapsulation element taking delivery of a segment of a transfer take delivery of every segment of that transfer: reassembly state is held by the element that takes delivery ({{reassembly}}), and an element holding only some of a transfer's segments can never complete it. A singleton endpoint satisfies this requirement trivially. An endpoint in a scheme whose delivery procedures deliver every bundle to every member ({{model}}) also satisfies it: each member takes delivery of the complete set of segments and reassembles independently. What cannot satisfy it is a multi-member endpoint for which the choice of delivering member is unspecified: distinct encapsulating bundles of one transfer may then be delivered to different members, each member accumulates a disjoint, incomplete set of segments, and the transfer fails silently at its expiry. Accordingly, an encapsulating node MUST NOT segment a bundle toward a decapsulation endpoint unless it is known — by the rules of the EID's scheme (every 'ipn' scheme EID identifies a singleton endpoint, {{Section 5.1 of ?RFC9758}}) or by configuration — that every bundle destined for that endpoint is delivered either to a single member or to every member. Whole-bundle payloads are not subject to this constraint: any member taking delivery holds the entire encapsulated bundle, although delivery to more than one member duplicates the encapsulated bundle at each.

## Transfer IDs {#transfer-ids}

Transfer IDs are unsigned integers that identify a transfer uniquely within the scope of the pair (source node ID, destination EID) borne by the primary blocks of the transfer's encapsulating bundles ({{primary-block}}). Both elements of the scope are explicit in every encapsulating bundle and are compared literally, as EIDs, at the decapsulating node ({{reassembly}}), so neither peer requires any context beyond the bundle itself — in particular, no knowledge of which node IDs name the same node, of how many encapsulation functions operate at a node, or of how many endpoints a decapsulation element is registered in. A node holding several node IDs ({{Section 4.2.5.2 of RFC9171}}) consequently has an independent Transfer ID space for each (source node ID, destination EID) pair it uses. A Transfer ID carries no ordering semantics: a decapsulation element MUST NOT infer transmission order, recency, or loss from the numeric values of Transfer IDs. There is no analog of the transfer window of {{BTPU}}: the temporal bound on a transfer is provided by bundle expiry, since the Bundle Protocol guarantees that no bundle is delivered after the end of its lifetime.

An encapsulating node MUST NOT assign a Transfer ID that identifies another transfer with the same (source node ID, destination EID), any of whose encapsulating bundles have yet to expire. Because every encapsulating bundle of a transfer shares a single expiry time ({{primary-block}}), a Transfer ID becomes safely reusable at exactly the transfer's expiry time. Subject to this rule, the assignment strategy is an implementation matter, and uniqueness in any wider scope implies uniqueness in this one: a sequential counter per (source node ID, destination EID) pair, a counter per destination EID, and a single counter for the whole node are all valid disciplines, as is random assignment. To satisfy the rule across restarts of the encapsulating node, an implementation SHOULD either persist sufficient assignment state or select Transfer IDs after restart in a manner unlikely to collide with transfers still in flight (e.g. derived from a timestamp). [^tidsize]

[^tidsize]: CBOR uints are variable-length with no fixed wrap point, and the encoding stays small (1–5 octets) for realistic values, so there is no pressure to keep the identifier space compact.

## Reception {#reception}

Upon delivery of the payload of an encapsulating bundle to a decapsulation element:

- If the payload is a whole-bundle payload, the encapsulated bundle SHALL be presented to the BPA as a received bundle — the decapsulation element here acting in the capacity of a convergence-layer adapter ({{model}}) — whereupon reception proceeds as defined in {{Section 5.6 of RFC9171}}: the encapsulated bundle may be forwarded, delivered, etc. Where the decapsulated data is not a BPv7 bundle (see {{use-cases}}), its disposition is determined by examination of its initial octets and is an implementation matter — e.g. presentation of a BPv6 bundle to a BPv6 agent.
- If the payload is a segment payload and the decapsulation element does not support reassembly, the element SHALL discard the payload. Support for reassembly is OPTIONAL; support for generating segments is likewise OPTIONAL.
- If the payload is a segment payload and the decapsulation element supports reassembly, the payload SHALL be processed as described in {{reassembly}}.

## Reassembly {#reassembly}

A decapsulation element maintains a reassembly structure per active transfer, keyed by the triple (source node ID, destination EID, transfer-id) of the encapsulating bundle. Every component of the key is taken from the encapsulating bundle itself, so the key is unambiguous even when one decapsulation element is registered in several endpoints. Source node IDs and destination EIDs are compared literally, as EIDs: a decapsulation element is not required to determine whether two distinct EIDs name the same node or endpoint, and the consistency rules of {{primary-block}} ensure that literal comparison suffices. The nature of the reassembly structure is an implementation matter.

Upon processing a segment payload:

- If segment-offset plus the length of encapsulated-data exceeds total-length, the payload is malformed: the decapsulation element SHALL discard it and SHOULD discard the reassembly structure for the transfer, if any.
- If a reassembly structure for the transfer exists and the payload's total-length differs from the transfer's previously recorded total-length, the transfer is corrupt: the decapsulation element SHALL discard the payload and the reassembly structure.
- The BP network may duplicate encapsulating bundles (multi-path forwarding, repetition at lower layers), so identical duplicates are normal and benign: if the octets of the segment duplicate octet ranges already received with identical content, the duplicate octets MAY be ignored. If they overlap previously received ranges with differing content, the transfer is corrupt: the decapsulation element SHALL discard the payload and the reassembly structure.
- Otherwise the decapsulation element SHALL insert the segment into the reassembly structure.
- If, after insertion, every octet in the range zero to total-length has been received, reassembly is complete: the reassembled encapsulated bundle SHALL be presented to the BPA as a received bundle ({{model}}), whereupon reception proceeds as defined in {{Section 5.6 of RFC9171}}, and the reassembly structure SHALL be discarded.

## Reassembly State {#resources}

A decapsulation element retains reassembly state for a transfer only while the transfer can still complete. An incomplete transfer MUST be discarded no later than the transfer's expiry — the single expiry time shared by all encapsulating bundles of the transfer ({{primary-block}}) — both because no further segment can arrive after that time, and because the Transfer ID may thereafter be reused for a new transfer ({{transfer-ids}}): reassembly state retained beyond expiry could wrongly absorb segments of the new transfer.

Before that deadline, retention of reassembly state is a matter for the same local resource management policies that govern any other storage the node commits. A transfer whose state is discarded under such policy has simply failed: the encapsulated bundle is never presented to the BPA. A configured upper bound on the interval for which an incomplete transfer is retained is one such policy, and is specifically recommended in {{resource-exhaustion}}.

# The Encapsulating Bundle {#encap-bundle}

An encapsulating bundle is a new bundle, originated by the encapsulating node acting in the capacity of a bundle source; it is not a modified copy of the encapsulated bundle, nor a forwarding of it. No field of the encapsulated bundle's primary block, and no extension block of the encapsulated bundle, is copied to the encapsulating bundle by any rule of this document: the encapsulated bundle contributes octets to the payload, and nothing else.

The converse also holds: the tunnel contributes nothing to the encapsulated bundle. The encapsulated bundle is immutable while encapsulated — the octets presented to the BPA of the decapsulating node ({{reception}}, {{reassembly}}) SHALL be exactly the octets captured by the encapsulation function ({{transmission}}); neither the encapsulation function nor the decapsulation element modifies them in any way, and any divergence is corruption ({{reassembly}}) or an attack ({{security-considerations}}), not processing. This invariance is what allows integrity protection applied to the encapsulated bundle itself to survive the tunnel intact.

An encapsulating bundle is an ordinary bundle, and its payload is not an administrative record; a decapsulation endpoint MUST NOT be the administrative endpoint of the node. The destination endpoint of an encapsulating bundle is a decapsulation endpoint, which is distinct from the destination of the encapsulated bundle. [^adminrec]

[^adminrec]: Carrying the payload as ordinary application data at a dedicated endpoint, rather than as an administrative record, avoids any administrative record type allocation, permits multiple decapsulation endpoints per node (e.g. per-tunnel), and keeps the administrative endpoint free of high-volume traffic.

## Primary Block {#primary-block}

The fields of the encapsulating bundle's primary block ({{Section 4.3.1 of RFC9171}}) are populated as for any newly created bundle, subject to the following:

Destination EID:
: SHALL be the decapsulation endpoint ID toward which the encapsulated bundle is being forwarded. When the payload is a segment payload, every encapsulating bundle of the transfer SHALL bear the same destination EID: the destination is one element of the scope that identifies the transfer ({{transfer-ids}}).

Source node ID:
: SHOULD be a node ID of the encapsulating node — the EID of any singleton endpoint in which that node is registered ({{Section 4.2.5.2 of RFC9171}}): anonymous (dtn:none) sourcing defeats the peer authentication measures described in {{security-considerations}}. Note that a node may legitimately hold many node IDs; which of them is used is a matter of policy. When the payload is a segment payload, however, the source node ID MUST NOT be dtn:none, and every encapsulating bundle of the transfer SHALL bear the same source node ID: a decapsulation element identifies a transfer by literal comparison of source node IDs ({{reassembly}}), without resolving which node an EID names, so a node holding several node IDs must use one of them consistently throughout a transfer.

Report-to EID:
: Set according to the policy of the encapsulating node.

Bundle processing control flags:
: Set according to the policy of the encapsulating node; the flags of the encapsulated bundle have no bearing on them. The "bundle's payload is an administrative record" flag MUST NOT be set.

Creation timestamp:
: Set as for any newly created bundle ({{Section 4.2.7 of RFC9171}}); it is unrelated to the creation timestamp of the encapsulated bundle.

Lifetime:
: SHALL be set such that the expiry time of the encapsulating bundle (creation time plus lifetime) is no later than the expiry time of the encapsulated bundle: the tunnel can never extend the transmission opportunity of the encapsulated bundle — an encapsulating bundle that outlived it would waste network resources carrying data already expired at decapsulation — and this bound underpins the transfer machinery of {{transfer-ids}} and {{resources}}. The expiry time MAY be earlier than that of the encapsulated bundle, deliberately curtailing the tunnel transit: for example, where it is known that a bundle failing to complete the tunnel sub-path within a certain duration cannot achieve onward delivery within its own lifetime. Where the encapsulated data has no expiry known to the encapsulating node (e.g. arbitrary binary data, {{use-cases}}), an expiry bound SHALL be established by a-priori configuration and applied in the same way.

For a segmented transfer, every encapsulating bundle of the transfer SHALL share a single expiry time — the transfer's expiry — regardless of when each is created, so that the expiry of a transfer is well defined at both the encapsulating and decapsulating nodes. Since a bundle's expiry is a quantity derived from its creation time and lifetime ({{Section 5.5 of RFC9171}}), this is achieved by a rolling calculation: each encapsulating bundle is created with its natural creation timestamp, and its lifetime is set to the interval remaining between its creation and the transfer's expiry. A node without an accurate clock performs the same calculation against locally elapsed time, setting each encapsulating bundle's lifetime to the transfer duration remaining at that bundle's creation and carrying a Bundle Age block as usual ({{Section 4.4.2 of RFC9171}}). [^rolling]

[^rolling]: The alternative — assigning all encapsulating bundles of a transfer an identical creation timestamp — is not viable in general: the sequence number component of the creation timestamp is generated by the source node's BPA as a property of the timestamp mechanism ({{Section 4.2.7 of RFC9171}}), not assignable per transfer, and duplicate (source node ID, creation timestamp) pairs risk the "unexpected network behavior" cautioned against there. The rolling calculation composes with any timestamp generator, and has the useful side effect that later-created encapsulating bundles carry shorter lifetimes, so the network never retains one longer than the transfer it serves.

Selection of all other parameters governing the forwarding of encapsulating bundles (priority, security policy, etc.) is likewise a matter for the encapsulating node; the corresponding parameters of the encapsulated bundle MAY be consulted where they are known.

## Extension Blocks

The encapsulating bundle carries only extension blocks pertaining to its own conveyance, inserted according to the policy of the encapsulating node and processed on the encapsulating path exactly as for any other bundle. In particular:

- A Hop Count block ({{Section 4.4.3 of RFC9171}}), if present, is new: its count starts at zero at the encapsulating node. The hop count of the encapsulated bundle, if any, does not advance while the bundle is encapsulated — the entire tunnel transit appears to the encapsulated bundle as a single hop. This is both a feature (tunnel transparency) and a hazard; see the loop considerations in {{deployment}}.
- A Bundle Age block ({{Section 4.4.2 of RFC9171}}), if present, reflects the age of the encapsulating bundle only.
- A Previous Node block ({{Section 4.4.1 of RFC9171}}) is inserted and replaced hop by hop on the encapsulating path in the normal manner, identifying nodes of the tunnel path, not of the encapsulated bundle's path.

Note that, symmetrically, the extension blocks of the encapsulated bundle are not processed while it is encapsulated. In particular, the Bundle Age block of an encapsulated bundle is not updated during tunnel transit, so the age of an encapsulated bundle originated by a node without an accurate clock is under-counted by the tunnel transit time. This is an unavoidable consequence of carrying the encapsulated bundle opaquely; the lifetime rule of {{primary-block}} bounds the error to at most the remaining lifetime of the encapsulated bundle at encapsulation. The decapsulation element cannot correct the encapsulated bundle's Bundle Age block: doing so would modify the encapsulated bundle, violating the opacity of the tunnel and invalidating any integrity protection covering it. Once the encapsulated bundle has been presented to the BPA of the decapsulating node, however, tunnel opacity is no longer at stake: the bundle is processed as any received bundle, and the BPA MAY account for the tunnel transit time in the Bundle Age block in the manner of {{Section 4.4.2 of RFC9171}}, exactly as a receiving node may account for known transmission time over any other convergence layer. The tunnel transit time may be known a priori (e.g. from a contact plan) or measured from the encapsulating bundles themselves: for a whole-bundle payload, the age of the encapsulating bundle at delivery — computed from its creation timestamp, or carried in its own Bundle Age block — is precisely the tunnel transit time, and for a segmented transfer the interval from the earliest creation timestamp among the transfer's encapsulating bundles to the completion of reassembly serves the same purpose.

The state in which the encapsulated bundle's blocks are frozen is the state in which the encapsulating node forwards it ({{transmission}}). A Previous Node block in a decapsulated bundle consequently identifies the encapsulating node: the single hop that the tunnel presents, consistent with {{diagnostics}}. The decapsulating node takes this block as it finds it — it cannot verify it in general (the encapsulating node may hold several node IDs, the encapsulated data may not be a BPv7 bundle, and the block is optional) and correcting it would modify the encapsulated bundle — but a stale or absent Previous Node block is consulted only during local processing at the decapsulating node and is replaced in the normal manner upon onward forwarding.

## Payload {#payload-format}

The payload of an encapsulating bundle SHALL be represented as a CBOR array, encoded with definite length. The array size discriminates the payload form:

- An array of size 1 is a whole-bundle payload. Its sole element is the encapsulated-data field.
- An array of size 4 is a segment payload: the three segmentation fields, in the order defined below, followed by the encapsulated-data field.
- All other array sizes are reserved for future use; a payload with a reserved array size MUST be discarded by a receiving decapsulation element.

The fields are defined as follows:

transfer-id:
: An unsigned integer identifying the transfer to which this segment belongs ({{transfer-ids}}).

total-length:
: An unsigned integer giving the total length in octets of the encapsulated bundle.

segment-offset:
: An unsigned integer giving the offset in octets, from zero, of the first octet of encapsulated-data within the encapsulated bundle.

encapsulated-data:
: A byte string, encoded with definite length. In a whole-bundle payload, its content is the entire encapsulated bundle; in a segment payload, its content is a contiguous sequence of octets of the encapsulated bundle.

An informative CDDL expression of this structure is provided in {{cddl}}.

# Deployment Considerations {#deployment}

The following characteristics of BIBE are to be considered before deployment:

1. It is unreliable: there is no BIBE-layer acknowledgement or retransmission. Where reliability is required it is expected to be provided by the convergence layers carrying the encapsulating bundles, by replication/repetition strategies at lower layers (see, e.g., {{BTPU}}), by custody or other acknowledgement signalling operating at a higher layer over the encapsulated bundles, or end to end by the communicating applications.
1. It fails silently: no condition at or beyond the decapsulation endpoint is signalled to the encapsulating node. A segment payload discarded because the decapsulation element does not support reassembly ({{reception}}), a transfer that expires or is discarded incomplete ({{resources}}), and a payload discarded for a reserved array size ({{payload-format}}) are, to the encapsulating node, indistinguishable from loss of the encapsulating bundles in transit — so a misconfigured tunnel, such as a segmentation threshold set toward a peer that does not support reassembly, presents as persistent silent loss. This document deliberately defines no signal reporting the disposition of a payload after delivery; the visibility available from existing status report machinery is set out in {{diagnostics}}, and verifying that a tunnel is correctly configured is an operational matter for its administrators. [^nosignal]
1. Encapsulating and decapsulating nodes require compatible configuration (the decapsulation endpoint, the segmentation threshold, and whether reassembly is supported), established in advance.
1. An encapsulating bundle whose destination endpoint has no registered decapsulation element at the destination node will be delivered to whatever application is registered there, or abandoned; nothing distinguishes an encapsulating bundle from any other bundle in transit.
1. As with IP tunnels, recursive encapsulation loops are possible: if the route toward a decapsulation endpoint itself resolves to BIBE encapsulation (at this node or a mutually recursive one), each encapsulation produces a new bundle that is encapsulated again, without limit. Unlike IP, a bundle has no hop-limit field that survives encapsulation, so the loop is not self-terminating within bundle lifetime. An implementation MUST protect against this, for example by refusing to encapsulate a bundle it recognizes as one of its own encapsulating bundles, by bounding nesting depth (compare the tunnel encapsulation limit of {{TUNLIMIT}}), or by configuration validation ensuring the route to a decapsulation endpoint does not traverse the tunnel it serves.

[^nosignal]: A post-delivery disposition signal would make every decapsulation element a bundle source, require an enumeration of failure reasons, and amount to the acknowledgement layer this protocol does not have; and the conditions it could report are configuration errors of a bilaterally configured tunnel, properly detected by its operators rather than corrected by the protocol.

## Diagnostics {#diagnostics}

BIBE requires no diagnostic machinery of its own: because encapsulating bundles are ordinary bundles and decapsulation ends in ordinary reception ({{Section 5.6 of RFC9171}}), the bundle status report mechanism of {{Section 6.1 of RFC9171}} already observes every stage of the encapsulation chain. Status reports are requested per bundle and honoured subject to node policy; they are a diagnostic instrument, and the report flags of encapsulating bundles SHOULD NOT be set in normal operation — a stream of status reports directed at the encapsulating node both amplifies traffic and identifies tunnel traffic to on-path observers ({{security-considerations}}).

When enabled for diagnosis, the observation points are:

- Tunnel ingress: encapsulation is a forwarding operation ({{model}}), so an encapsulated bundle that requests forwarding reports causes a "bundle forwarded" status report from the encapsulating node in the normal way.
- Tunnel transit: the encapsulating node sets the report flags and report-to EID of the encapsulating bundles it sources ({{primary-block}}). Reception and forwarding reports trace the tunnel path; a deletion report identifies where and why an encapsulating bundle was lost, using the existing reason codes (e.g. "Lifetime expired", "Depleted storage", "No known route to destination from here", "No timely contact with next node on route", "Block unintelligible", "Hop limit exceeded" — the latter being the signature of the encapsulation loops cautioned against in {{deployment}}).
- Transfer completion: delivery reports requested on the encapsulating bundles of a transfer confirm, chunk by chunk, that its payloads reached the decapsulation element; a delivery report still absent at the transfer's expiry ({{primary-block}}) identifies the lost segment(s) to the encapsulating node without any signal from the decapsulating peer.
- Tunnel egress: the reassembled or whole encapsulated bundle is presented to the BPA per {{Section 5.6 of RFC9171}}, so its own reception, deletion, and onward forwarding reports resume in the normal way; to the encapsulated bundle's reporting, the entire tunnel transit appears as a single hop.

These observation points compose: delivery reports confirming that every encapsulating bundle of a transfer reached the decapsulation endpoint, while the encapsulated bundle's own reception report from the decapsulating node never arrives, localize the failure to the decapsulation element itself — a configuration mismatch ({{deployment}}) rather than transit loss — which is precisely the distinction an administrator needs when a tunnel fails silently.

The dispositions internal to the decapsulation element ({{reception}}, {{reassembly}}, {{resources}}) occur after delivery of the encapsulating bundle and concern data that is not (or not yet) a bundle; they are not expressible as status reports about either bundle, and SHOULD instead be surfaced by local means (logs, counters) at the decapsulating node.

# Security Considerations {#security-considerations}

Security considerations largely follow those of BPv7 {{!RFC9171}} and BPSec {{?RFC9172}}. In addition, because BIBE is a tunneling protocol, the security concerns cataloged for IP tunneling in {{TUNSEC}} apply, translated to the BP layer, and are addressed in the following subsections.

## Composition with BPSec

BIBE composes constructively with BPSec: a BCB on the payload of the encapsulating bundle encrypts the entire encapsulated bundle, including its primary block, concealing the encapsulated source and destination from observers on the encapsulating path.

## Circumvention of Policy Enforcement

As {{TUNSEC}} observes for IP, tunneled traffic is opaque to inspection and filtering points along the tunnel path: a node on the encapsulating path that would, by policy, have refused to forward or store the encapsulated bundle (by destination, source, block content, size, or any other criterion) cannot apply that policy, particularly when the encapsulating payload is encrypted. BIBE therefore permits bundles to cross policy boundaries that they could not cross natively — which is precisely its cross-domain use case when authorized, and a policy-evasion channel when not.

Consequently the decapsulating node is a policy enforcement point: a decapsulated bundle MUST be subjected to the node's normal inbound bundle admission policy, exactly as if it had arrived via any convergence layer, before it is forwarded or delivered. A node SHOULD NOT afford decapsulated bundles any trust derived from the encapsulating bundle's security verification unless policy explicitly says so. Network operators SHOULD restrict, by policy, where decapsulation elements may be registered and from which peers each will accept encapsulating bundles, just as prudent IP networks control where tunnels may terminate.

## Spoofing of the Encapsulated Bundle

Nothing in this protocol authenticates the contents of an encapsulated bundle: the source of the encapsulated bundle is whatever its primary block claims, and an authorized (or compromised) encapsulating peer can inject bundles claiming any source. Authentication of the encapsulating bundle (e.g. a Block Integrity Block (BIB) covering the payload) authenticates only the tunnel peer, not the encapsulated bundle's provenance; end-to-end assurance of the encapsulated bundle requires BPSec applied to the encapsulated bundle itself.

## Injection Against Transfers {#injection}

An adversary able to cause a fabricated bundle to be delivered at a decapsulation endpoint, bearing the (source node ID, destination EID) pair and Transfer ID of a transfer in progress, can attack that transfer. An inconsistent forged segment — one with a mismatched total-length, or content overlapping previously received ranges — causes the reassembly rules of {{reassembly}} to discard the transfer; this is a targeted denial of service against a single transfer. Note, however, that the discard rules are not the vulnerability: the same adversary can instead inject a consistent forged segment — matching total-length, at an offset not yet received — and thereby corrupt the reassembled bundle silently, which no reassembly rule can detect. Injection of forged segments is fatal to a transfer whatever the discard rules do; the rules of {{reassembly}} merely convert the detectable subset of such corruptions into prompt failure, releasing reassembly state rather than expending further resources on a transfer that can no longer complete correctly.

The mitigation is the same as for resource exhaustion ({{resource-exhaustion}}): authenticate encapsulating bundles (e.g. with a BIB covering the payload) so that forged payloads are discarded before they reach the reassembly procedures, and note that integrity protection applied to the encapsulated bundle itself, surviving the tunnel intact ({{encap-bundle}}), detects any corrupt reassembly after decapsulation.

## Resource Exhaustion {#resource-exhaustion}

A decapsulating node is exposed to resource-exhaustion attack by an adversary transmitting incomplete transfers, or segment payloads with large total-length values. Although {{resources}} bounds the retention of reassembly state by the expiry of the encapsulating bundles, that expiry is claimed by the sender: an adversary can assign its encapsulating bundles distant expiry times and simply decline to send one or more segments. A decapsulating node SHOULD therefore be configurable with an upper bound of its own on the retention of reassembly state — a maximum interval for which an incomplete transfer may be held, or equivalently a maximum acceptable remaining lifetime for encapsulating bundles carrying segment payloads — and discard, as {{resources}} permits, any transfer exceeding it: the sender-claimed expiry then bounds well-behaved transfers, while the configured bound caps the exposure to hostile or broken ones. Because decapsulation endpoints are established by administrative policy rather than at a well-known service identifier ({{model}}), an adversary has no predictable target for such traffic; operators SHOULD treat decapsulation endpoint IDs as configuration shared only with authorized tunnel peers. The expiry deadline and local resource management policies of {{resources}} mitigate this; deployments SHOULD additionally authenticate encapsulating bundles (e.g. with a BIB covering the payload) so that payloads from unauthorized sources can be discarded before reassembly state is committed. Authentication narrows the exposure to authorized peers but does not eliminate it: a compromised or misbehaving authorized peer can still commit reassembly storage up to whatever bound local policy enforces, and that policy is therefore the mitigation of last resort.

Recursive encapsulation is a related amplification hazard; see {{deployment}} for the required loop protections.

# IANA Considerations {#iana-considerations}

This document has no IANA actions: decapsulation endpoints are established by administrative policy ({{model}}), so no well-known service identifier is defined, and the payload of an encapsulating bundle is not an administrative record, so no administrative record type is allocated.

--- back

# CDDL Expression {#cddl}

For informational purposes, {{fig-cddl}} restates the structure defined in {{payload-format}} in CDDL {{?RFC8610}}, extending the `$payload-block-data` socket defined in Appendix B of {{!RFC9171}}: the payload block content is an encoded CBOR array in which the three segmentation fields are optional as a group — either all present (a segment payload) or all absent (a whole-bundle payload) — and the final field is always a byte string. If this CDDL and the text definitions of {{payload-format}} disagree, the text governs.

~~~ cddl
$payload-block-data /= bstr .cbor encapsulation-payload

encapsulation-payload = [
  ? (
    transfer-id: uint,
    total-length: uint,
    segment-offset: uint
  ),
  encapsulated-data: bstr
]
~~~
{: #fig-cddl title="CDDL definition of the encapsulating bundle payload" }

Note that encoding constraints (definite-length encoding) are matters for the text definitions of {{payload-format}}; CDDL does not express encoding options. Note also that encapsulated-data is deliberately a plain bstr rather than `bstr .cbor bundle`: its content may be a BPv6 bundle (which is not CBOR) in the version-encapsulation use case, and in a segment payload it is an arbitrary slice of octets that need not be well-formed CBOR at all.

# Acknowledgments
{:numbered="false"}

This document is directly derived from draft-ietf-dtn-bibect, authored by Scott Burleigh, Alberto Montilla, Joshua Deaton, and Carlo Caini; its architecture and concepts originate with them. The central insight — that a BP network can itself serve as the "link layer" beneath a further layer of bundle protocol — is Scott Burleigh's, and the encapsulation-only scope of this document is a return to that of his original 2013 formulation, draft-irtf-burleigh-bibe.

The segmentation mechanism was designed within the CCSDS Bundle-in-Bundle Encapsulation effort of the Space Internetworking Services (SIS-DTN) working group, which identified bundle segmentation, rather than BPv7 fragmentation, as the appropriate tool for carrying large bundles over size-limited paths, and defined the representation this document adopts. Joshua Deaton contributed to both the IETF and CCSDS strands of that work, which was demonstrated in a prototype implementation within DTNME at NASA. The segmentation model is aligned with the transfer/segment design of {{BTPU}}.

Brian Sipos contributed the CDDL formulation in {{cddl}}, the discipline of keeping normative text definitions strictly separate from CDDL, and the review that led to the PDU-derived Transfer ID scope of {{transfer-ids}}.

The original bundle-in-bundle encapsulation concept derives from draft-irtf-dtnrg-bundle-encapsulation (2009) by Susan Symington, Bob Durst, and Keith Scott of The MITRE Corporation, whose influence is gratefully acknowledged.
