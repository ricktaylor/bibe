# BIBE: Bundle Aggregation and the Payload Encoding

**Status:** Discussion note. Not Working Group text, and not yet in any revision of the draft.<br>
**Relates to:** [draft-ietf-dtn-bibe](https://ricktaylor.github.io/bibe/draft-ietf-dtn-bibe.html). References of the form "draft §N" are to sections of the current editor's copy; plain "section N" refers to this note.<br>
**Author:** Rick Taylor<br>
**Date:** 15 November 2026 (fourth version; earlier versions 22 October, 11 November and 12 November 2026)

## 1. Summary

This note proposes adding bundle aggregation to BIBE: carrying several complete bundles, headed for the same decapsulation endpoint, in one encapsulating bundle, so that they share the cost of its primary block and any BPSec blocks. Felix Flentge suggested it, as the counterpart to segmentation, on the DTN list on 28 September 2026.

Discussion of the earlier versions has settled several principles:

- **One encoding for each kind of payload.** A single bundle, an aggregate and a segment should each have exactly one encoding. Keeping the current forms and adding typed equivalents alongside them would leave receivers, including on-board implementations, supporting both for decades (section 7.3).
- **Aggregation is optional** for senders, and bundles that need different treatment are never combined.
- **BIBE aggregation complements application-layer aggregation.** It works at intermediate nodes, on bundles from many sources, which an application-layer service cannot (section 7.2).

That leaves one decision: how to encode an aggregate. There are two options, set out side by side in section 3:

- **Option A: every payload is a [type, content] pair.** Cleanest, and extensible, but not compatible with the current format.
- **Option B: an aggregate inside the current 1-element form.** Fully compatible with the current format, with no type codes.

The rules for aggregation itself (section 4) are the same under either option.

## 2. The current format

Today (draft §5.3) the payload is a definite-length CBOR array, and its length identifies the form:

| Array length | Form | Contents |
|:---:|---|---|
| 1 | Whole-bundle payload | `[encapsulated-data]` |
| 4 | Segment payload | `[transfer-id, total-length, segment-offset, encapsulated-data]` |
| other | Reserved | Discarded by the receiver |

This format comes from the CCSDS BIBE draft, a draft experimental specification that was never published as a CCSDS standard. It has been implemented in several prototypes, and an implementation is expected in a future ION release.

Identifying the form by array length works for two fixed forms. Aggregation is the first variable-length form, and a flat list of bundles would collide with the segment form: an aggregate of four bundles would have array length 4. There are two ways to add it without ambiguity: give every payload an explicit type code alongside its content (option A), or nest the list of bundles inside the 1-element form (option B).

## 3. The two options

### 3.1 Option A: every payload is [type, content]

Every payload is a definite-length CBOR array of two elements: an unsigned integer **payload type**, then the **content**, whose structure the type defines. The type, not the array length, says how to read the content:

| Type | Name | Encoding | Content |
|:---:|---|---|---|
| 0 | Segment | `[0, [transfer-id, total-length, segment-offset, segment-data]]` | The four fields of the current segment form |
| 1 | Bundle | `[1, encapsulated-bundle]` | One complete bundle, as a byte string |
| 2 | Aggregate | `[2, [+ encapsulated-bundle]]` | One or more complete bundles, each as a byte string |

The content of types 0 and 1 is exactly the corresponding element or elements of the current format, so an implementation can reuse its existing parsing for them.

An aggregate SHALL contain at least one bundle. An aggregate of one bundle, `[2, [bstr]]`, is valid, and is processed exactly as `[1, bstr]`; a sender SHOULD use type 1, which is one octet shorter.

A receiver discards a payload that is not a 2-element array, whose first element is not an unsigned integer, whose type it does not recognize, or whose content does not match what its type requires. Future types are allocated from a registry (section 5).

**Encodings.** Encapsulated bundle octets are abbreviated as `9f 89 …`: a BPv7 bundle begins with an indefinite-length array (`9f`) whose first item is the primary block (`89`, an array of 9 items).

```
82 01 58 3c 9f 89 …                       single 60-octet bundle  [1, bstr]
82 02 83 58 28 9f 89 … 58 34 … 58 78 …    aggregate of 40, 52 and 120 octets  [2, [b, b, b]]
82 00 84 07 1a 00 01 86 a0 00 59 04 00 …  segment: transfer 7, total 100000, offset 0, 1024 octets  [0, [7, 100000, 0, bstr]]
```

**CDDL:**

```cddl
$payload-block-data /= bstr .cbor encapsulation-payload

encapsulation-payload = $encapsulation-payload

$encapsulation-payload /= segment-payload
$encapsulation-payload /= bundle-payload
$encapsulation-payload /= aggregate-payload

segment-payload = [
  payload-type: 0,
  segment: [
    transfer-id: uint,
    total-length: uint,
    segment-offset: uint,
    segment-data: bstr
  ]
]

bundle-payload = [
  payload-type: 1,
  encapsulated-bundle: bstr
]

aggregate-payload = [
  payload-type: 2,
  encapsulated-bundles: [ + bstr ]
]
```

### 3.2 Option B: an aggregate inside the current format

The two current forms are unchanged. An aggregate is a 1-element array whose single element is an array of bundles, rather than a byte string:

| Array length | Element | Form |
|:---:|---|---|
| 1 | byte string | Single bundle `[bstr]`, unchanged |
| 1 | array | **Aggregate** `[[bstr, bstr, …]]` |
| 4 | | Segment `[transfer-id, total-length, segment-offset, bstr]`, unchanged |

An aggregate SHALL contain at least one bundle. A receiver discards an empty aggregate, or one with any element that is not a byte string, as it already discards any payload that does not conform (draft §5.3). An aggregate of one bundle, `[[bstr]]`, is valid, and is processed exactly as the single-bundle form `[bstr]`; a sender SHOULD use the single-bundle form, which is one octet shorter, and which a receiver of the current format also accepts.

A receiver tells a single bundle from an aggregate by the CBOR major type of the 1-element array's only element: a byte string (major type 2, initial octet `40`–`5f`) is a single bundle, and an array (major type 4, initial octet `80`–`9f`) is an aggregate. This is one check on one octet, made only for 1-element payloads, after the array length has been read.

**Encodings:**

```
81 58 3c 9f 89 …                        single 60-octet bundle  [bstr]  (unchanged)
81 83 58 28 9f 89 … 58 34 … 58 78 …     aggregate of 40, 52 and 120 octets  [[b, b, b]]
84 07 1a 00 01 86 a0 00 59 04 00 …      segment: transfer 7, total 100000, offset 0, 1024 octets  (unchanged)
```

**CDDL:**

```cddl
$payload-block-data /= bstr .cbor encapsulation-payload

encapsulation-payload = bundle-payload / aggregate-payload / segment-payload

bundle-payload = [
  encapsulated-bundle: bstr
]

aggregate-payload = [
  [ + encapsulated-bundle: bstr ]
]

segment-payload = [
  transfer-id: uint,
  total-length: uint,
  segment-offset: uint,
  segment-data: bstr
]
```

### 3.3 Comparison

| | Option A: [type, content] | Option B: nested aggregate |
|---|---|---|
| One encoding for each kind of payload | Yes, except that a single bundle may also be sent as an aggregate of one (section 3.1) | Yes, except that a single bundle may also be sent as an aggregate of one (section 3.2) |
| Compatible with the current format | No: every implementation must change | Yes: single bundles and segments unchanged |
| Receiving aggregates | Only receivers known to support them (section 4.1) | Only receivers known to support them (section 4.1) |
| How a receiver identifies the form | The type code | The array length, then, for 1-element arrays, the major type of the element |
| Future extension | New type codes from a registry | Reserved array lengths; a typed form could be added in one later if needed |
| IANA actions | A new registry, or fixed values (section 5) | None |
| Framing for a single bundle or segment | One octet more than today for a single bundle, two for a segment | Unchanged |
| Framing for an aggregate | One octet more than option B | One octet less than option A |

**The case for A.** Every payload is identified the same way, by a type code a receiver dispatches on without inspecting element types, which suits high-rate on-board parsers. The content of each type is the existing encoding, so existing parsing code carries over. New forms need only a type code. The cost is the break: every implementation of the current format must change, and peers must change together.

**The case for B.** Nothing that existing implementations send or accept changes, and there are no IANA actions. The cost is that a receiver must check one octet's major type to tell a single bundle from an aggregate, and that each new form must find its own unused combination of array length and element type, with its own special case in every parser. A typed form could be added in a reserved length if one is ever needed, but every receiver would then carry two ways of identifying forms for good.

### 3.4 Overhead

Aggregation saves the cost of the encapsulating bundle. A minimal encapsulating bundle costs about 52 octets beyond its payload. That assumes two-element ipn EIDs, a CRC-16 on the primary block, no CRC on the payload block, and no extension blocks. A BIB with a typical 384-bit HMAC adds about another 72 octets.

For bundles of 100 octets each, sent separately or together as one aggregate, the total octets sent, and the share of them that are encapsulated bundle octets, are as follows (option B; under option A, sending separately costs one octet more per bundle, and each aggregate costs one octet more):

| Bundles | Separately, no BPSec | Aggregated, no BPSec | Separately, with BIB | Aggregated, with BIB |
|:---:|:---:|:---:|:---:|:---:|
| 1 | 156 (64%) | n/a | 228 (44%) | n/a |
| 4 | 624 (64%) | 464 (86%) | 912 (44%) | 536 (75%) |
| 10 | 1560 (64%) | 1076 (93%) | 2280 (44%) | 1148 (87%) |
| 20 | 3120 (64%) | 2096 (95%) | 4560 (44%) | 2168 (92%) |

Most of the benefit comes from the security blocks. A deployment that protects every encapsulating bundle with a BIB, or encrypts it with a BCB, pays for that once per aggregate rather than once per small bundle. The same applies to a digital signature applied at an intermediate node.

## 4. Aggregation rules

These rules apply under either option.

### 4.1 Optional

Generating aggregates is OPTIONAL. A sender that never aggregates behaves exactly as today.

Under either option, a receiver that does not support aggregation discards an aggregate, so a sender SHALL NOT send an aggregate to a decapsulation endpoint unless it is known, by configuration, that the decapsulation element supports aggregates. This is the same kind of rule the draft applies to segmentation, which is used only toward a decapsulation element known to support reassembly (draft §4.1). A misconfiguration then shows up as persistent silent loss, as for a segmentation threshold set toward a peer that does not support reassembly (draft §6, item 2).

### 4.2 What may be aggregated

An encapsulation function MAY aggregate any bundles that it is forwarding by encapsulation toward the same decapsulation endpoint. The bundles may have different sources and destinations. A bundle in an aggregate may itself be an encapsulating bundle, so aggregation works with nesting.

An encapsulation function MUST NOT aggregate bundles for which the encapsulating node would choose different forwarding parameters, were each carried in an encapsulating bundle of its own. These parameters include priority and any class-of-service treatment, security policy (the BPSec blocks applied), and status report requests. Mixing them would give some bundles the treatment meant for others: time-critical bundles would wait for, and share the fate of, bulk traffic. In particular, near-real-time and non-real-time traffic, such as streaming data and file transfers, are never combined in one aggregate.

### 4.3 Size

The total of the encapsulated octets carried in one encapsulating bundle SHALL NOT exceed the limit configured for the endpoint.

That limit currently exists only for segmentation, as the "segmentation threshold" (draft §4.2). This proposal renames it the **maximum encapsulation size**, because it now limits aggregates too. A bundle larger than the limit is segmented as today. An aggregate is never segmented, and segments are never aggregated.

### 4.4 Lifetime

Felix Flentge proposed the following rule, which this note adopts:

> The expiration time of the encapsulating bundle SHALL NOT be later than the latest expiration time of the encapsulated bundles. For a segment, it SHALL NOT be later than the expiration time of the segmented bundle.

In the terms the draft now uses (draft §5.1), the lifetime of an encapsulating bundle carrying an aggregate SHALL NOT exceed the greatest remaining lifetime among the bundles it carries, and the rule for segments is unchanged.

So an aggregate lives as long as its longest-lived bundle, and one short-lived bundle does not cut short the others. A bundle in the aggregate may then expire in transit. It is not delivered onward: each bundle is presented to the BPA of the decapsulating node as a received bundle, and RFC 9171 deletes an expired bundle at any point in its processing (Section 5.5 of RFC 9171). The tunnel never delivers an expired bundle; the cost is carrying it part of the way. Senders will usually prefer to aggregate bundles with similar remaining lifetimes.

The draft's tighter rule exists mainly for segmented transfers, where expiry governs the reuse of Transfer IDs and the discard of reassembly state (draft §4.3 and §4.6). An aggregate has neither, so a longer lifetime is harmless.

### 4.5 Holding bundles to aggregate them

An encapsulation function MAY hold a bundle for a limited, configured time (the **maximum aggregation delay**) so that it can be aggregated with bundles presented later. It SHALL NOT hold a bundle beyond that time, nor beyond the bundle's own expiry.

The maximum aggregation delay SHOULD default to zero. With no hold time, an encapsulation function aggregates only bundles that are presented to it together, for example the bundles a BPA releases toward the tunnel when a contact opens, so aggregation adds no delay at all. On a link that waits for its next contact, such as a spacecraft between ground station passes, bundles accumulate anyway, and combining them when the contact opens costs nothing in latency. A node that does configure a hold time can set it per class of traffic: zero for expedited bundles, for example.

Whether a bundle is held deliberately or is simply waiting with others, there are three consequences for the rest of the draft:

- **When forwarding completes.** Forwarding of a held bundle is complete only when transmission of the encapsulating bundle carrying it is requested. Draft §4.1 currently ties completion to presentation, and would need to change. Until then the bundle is held like any bundle waiting for a forwarding opportunity, and it is deleted in the normal way if it expires.
- **When a bundle is encapsulated.** Draft §4.1 requires a bundle to be encapsulated as it stands after forwarding-time processing, including the advance of any Bundle Age block. For a held bundle, that processing, and so the encapsulation, happens when the aggregate is formed, not when the bundle is first presented.
- **Diagnostics.** A held bundle that requests forwarding status reports gets its "bundle forwarded" report when the aggregate is transmitted (draft §6.1).

The draft would not specify how long to hold bundles, how many to collect, or when to send an aggregate early. Those are implementation choices, limited only by the maximum aggregation delay.

### 4.6 Reception

Each bundle in an aggregate SHALL be presented to the BPA as a separate received bundle, exactly as a whole-bundle payload is presented today (draft §4.4). The order in which they are presented is not specified: BPv7 makes no ordering guarantee anywhere else, and an aggregate is no different. Each then goes through reception processing and the node's inbound admission policy (draft §7.3) separately. A malformed bundle is disposed of by that processing and does not affect the others.

The payload as a whole must still conform to the format before any bundle is presented. BPv7 delivers a payload whole or not at all, so a payload that does not conform indicates a faulty or hostile sender, and the draft already requires it to be discarded (draft §5.3). The safe response is to discard it whole rather than present the bundles that happen to parse.

Aggregation keeps no state across payloads, and has no counterpart to Transfer IDs or reassembly.

### 4.7 Shared loss

Losing an encapsulating bundle loses every bundle it carries. Draft §6 should say so: aggregation reduces overhead, but it also makes losses larger. Where some bundles must be protected from that, they can be left out of aggregates (section 4.2), or protected with custody or acknowledgement at a higher layer, as draft §6 item 1 already suggests.

### 4.8 Custody and status reporting

Aggregation changes nothing here. Each bundle in an aggregate is an encapsulated bundle like any other: its own extension blocks, including any custody transfer or status reporting blocks, pass through the tunnel unchanged (draft §5) and take effect when it is presented to the BPA at the decapsulating node. The encapsulating bundle carrying an aggregate is subject to the same rules as any other encapsulating bundle. Custody and reporting apply to an aggregate exactly as they apply to a single encapsulated bundle or to a segment.

### 4.9 Anycast

Aggregates are not subject to the segmentation constraint for multi-member and anycast endpoints in draft §4.2: whichever member takes delivery has every bundle in the aggregate. So aggregation works without restriction for the anycast downlink case discussed on the list.

## 5. Payload type registry (option A only)

The draft currently has no IANA actions, and the reasons still hold for the service number and the administrative record type. Under option A, the payload type code exists to allow extension, and a registry is the normal way to manage that.

Proposed: a new **"BIBE Payload Types"** registry in the "Bundle Protocol" registry group. Values 0 to 23 encode in one octet, so they are the most valuable.

| Values | Registration procedure |
|:---:|---|
| 0 | Segment (this document) |
| 1 | Bundle (this document) |
| 2 | Aggregate (this document) |
| 3–19 | Specification Required |
| 20–23 | Experimental Use |
| 24 and above | Specification Required |

Specification Required, rather than Standards Action, would let CCSDS Recommendations allocate types without needing an IETF document.

The alternative is to define types 0, 1 and 2 in the document and reserve all other values for documents that update it. That keeps the draft free of IANA actions, but every extension would then need an update to the BIBE RFC itself.

Option B needs no IANA actions.

## 6. Changes to the rest of the draft

| Draft section | Option | Change |
|---|:---:|---|
| Abstract, 1.1 Use Cases | Both | Add aggregation as a use case: spreading the cost of the encapsulating bundle and its security blocks across many small bundles. |
| 2.1 Terminology | Both | Define "aggregate", "aggregation" and "maximum encapsulation size". |
| 4.1 Transmission | Both | Allow holding bundles, with no hold by default (section 4.5). Change when forwarding completes, and the point at which held bundles are encapsulated. Send aggregates only toward peers known to support them. |
| 4.2 Segmentation | Both | Rename the segmentation threshold to the maximum encapsulation size. State that aggregates are never segmented. |
| 4.4 Reception | Both | Present each bundle of an aggregate separately (section 4.6). |
| 5.1 Primary Block | Both | Add the lifetime rule for aggregates (section 4.4). |
| 5.3 Payload | A | Replace with section 3.1. |
| 5.3 Payload | B | Add the aggregate form of section 3.2. |
| 6 Deployment | Both | Add shared loss (section 4.7) and the delay added by holding bundles. Add support for aggregates to the configuration items, and to item 2 (failing silently). |
| 6.1 Diagnostics | Both | Describe when held bundles get "bundle forwarded" reports, and that one deletion report for an encapsulating bundle covers every bundle it carried. |
| 7 Security | Both | An aggregate protected by a BCB hides the number, sizes and boundaries of the bundles inside it from on-path observers, which strengthens the traffic-analysis protection of draft §7.1. Aggregation adds no reassembly state, so it gives an attacker no new way to exhaust resources; each bundle is admitted on its own. |
| 8 IANA | A | Create the registry in section 5, or fix the values. |
| Appendix A | Both | Replace the CDDL with that of section 3.1 or 3.2. |

## 7. Alternatives considered

### 7.1 Why aggregation and segmentation belong in one service

Wherever traffic is carried through a tunnel or over a constrained link, aggregation and fragmentation turn up together, as two sides of the same function: fitting the units of one layer into the units of the layer that carries them.

- **IPsec.** RFC 9347 (IP Traffic Flow Security, Standards Track) defines a single "aggregation and fragmentation mode" for ESP tunnels. Small IP packets are packed into one ESP payload and large ones split across several, by the same mechanism and at the same tunnel endpoints.
- **Link and subnetwork layers.** IEEE 802.11 A-MSDU aggregation sits alongside MSDU fragmentation in the same MAC. Satellite encapsulations such as DVB GSE, and the CCSDS packet transfer services, both pack several small packets into a frame and split a large packet across frames.
- **LTP in ION.** Each LTP span in ION is configured with an aggregation size threshold and time limit, for packing bundles into a block, alongside a maximum segment size, for splitting that block into segments. The two are set in the same command, for the same neighbour.

They go together because their configuration is the same. Both need a peer at the other end, an agreement on the carrying layer's size limit, and the per-tunnel configuration that BIBE already requires (draft §6, item 3). Once that tunnel exists, aggregation needs one more parameter, the hold time, which defaults to zero (section 4.5).

Putting aggregation in a separate service would duplicate all of that: a second set of peers, a second set of endpoints, a second size limit to keep consistent with the first, and a second layer of encapsulation with its own overhead, which works against the very per-bundle overhead that aggregation exists to reduce. It would also make the two interact awkwardly. A bundle near the size limit might be aggregated by one service and then need segmenting by the other, with each service unaware of the other's decisions.

The draft already accepts segmentation in BIBE, in part because BPv7 fragmentation interacts poorly with BPSec. Aggregation is the other half of the same job, and it belongs in the same place.

### 7.2 Leaving aggregation to payload conditioning

Delay-Tolerant Payload Conditioning (DTPC) is a service that runs above BP. It is implemented in ION, specified in Annex E of CCSDS 734.2-B-1, and described in draft-johnson-dtn-dtpc-00. It aggregates the application data units (ADUs) of one application for one destination, end to end, together with ordering, acknowledgement and duplicate suppression. It has been suggested that aggregation belongs there rather than in BIBE, because once bundles have been created an encapsulating node no longer knows the application's requirements, such as ordered delivery.

The two do different jobs:

- **Scope.** Payload conditioning aggregates one application's data between two endpoints that are both set up for it. BIBE aggregation combines complete bundles from any sources, toward any destinations, over only the part of the path where it helps: one constrained link, or the crossing of one domain. By the time bundles from different sources meet at that point they are already bundles, and only something at the bundle layer can combine them.
- **Nothing the application needs is lost.** BIBE never modifies an encapsulated bundle (draft §5). Any extension blocks an application relies on pass through an aggregate unchanged, such as the sequence identification, custody transfer and compressed status reporting blocks proposed in draft-flentge-dtn-bundle-sequence-identification, draft-flentge-dtn-bpv7-custody-transfer and draft-flentge-dtn-compressed-status-reporting. Ordering is restored at the destination exactly as for bundles arriving by any other path. Unlike fragmentation, aggregation never splits an ADU or alters a bundle.
- **What aggregation does need is visible.** Whether bundles can be aggregated depends on their forwarding treatment and lifetime, which the encapsulating node can see: priority and class of service, BPSec policy, and expiry. Section 4.2 forbids combining bundles that differ in treatment, and section 4.5 makes no hold time the default, so aggregation neither adds delay nor mixes traffic with different needs.
- **Shared security blocks.** Only aggregation at the bundle layer lets one BIB, BCB or digital signature cover bundles from many different sources (section 3.4).

The two are complementary. An application that wants its own data aggregated can use payload conditioning, or build it into its own protocol, as was noted on the list for RTP over BP. Its bundles can then still be aggregated by BIBE across a constrained part of the path.

### 7.3 Current and typed forms side by side

The 11 November version of this note kept the current forms as a "legacy mode" and added typed equivalents, in the reserved array length 2, for new implementations to use instead. It was fully compatible, but it gave every kind of payload two encodings. Receivers would have had to support both indefinitely, since a deprecated form must still be accepted for as long as any sender might use it. For on-board implementations, which stay in service for decades, that is a lasting cost. Options A and B each give every kind of payload exactly one encoding.

### 7.4 Other encoding choices

- **A flat list of bundles, `[bstr, bstr, …]`**, identified by its first element being a byte string. An aggregate of four bundles would collide with the segment form, which also has four elements.
- **CBOR tags as the type code.** Tags would need allocations in the IANA CBOR Tags registry and cost 2–3 octets each. Some CBOR decoders accept unknown tags and strip them, so an implementation of the current format might not reject a tagged payload reliably.
- **A CBOR map.** It would allow optional fields, but costs more per payload, and nothing proposed needs optional fields.
- **Embedding bundles as CBOR rather than byte strings.** BPv7 bundles are self-delimiting CBOR, so this would save one header per bundle. But the decapsulation element would have to parse each bundle to find where it ends. That breaks the opacity of the encapsulated bundle, and one malformed bundle would make the rest unparsable.
- **Binary type-length-value framing, as in BTPU.** BTPU uses it because it fills fixed-size link-layer frames. BIBE payloads are already inside BPv7 and CBOR-encoded, so CBOR is the consistent choice.

## 8. Compatibility with existing implementations

Implementations of the current format, from the CCSDS draft, include DTNME's segmentation prototype, Lukas Holst's proof-of-concept, Aalyria's proof-of-concept, and an implementation expected in a future ION release. ION's current BIBE (the `bpv7/bibe` module of ION-DTN) implements the older draft-ietf-dtn-bibect format, which draft-ietf-dtn-bibe-00 already left behind. Unibo-BP's BIBE is being updated to the current draft, with segmentation, for June 2027.

**Under option A,** every implementation of the current format must change, and both ends of a tunnel must change together: each discards the other's payloads. The change itself is small, wrapping each existing encoding in a two-element array with a type code, and the draft is not yet published. It has been argued that changes of this kind are expected during standardisation, and that there are no operational deployments of the current format yet. Implementations not yet released, or still being written, could adopt the new format directly.

**Under option B,** nothing changes for existing implementations. They continue to send and accept single bundles and segments exactly as today. An implementation of the current format discards an aggregate, which does not conform to its format, which is why a sender may send aggregates only toward peers known to support them (section 4.1).

## 9. Open questions

1. **Option A or option B.** Is a clean, extensible format worth breaking the current one before publication, or should the current format be preserved?
2. **Registry policy (option A).** Is a registry with Specification Required right, or should the values be fixed in the document? Would CCSDS (via SANA) want its own range of values?
3. **Forwarding parameters.** Beyond priority, class of service, security policy and status reporting, which parameters matter when deciding whether bundles can be aggregated in ESA and CCSDS deployments?
4. **Naming.** "Aggregate", "maximum encapsulation size" and "maximum aggregation delay" are working names.

Comments are welcome on the DTN list (dtn@ietf.org) or as issues on the draft's repository, https://github.com/ricktaylor/bibe.
