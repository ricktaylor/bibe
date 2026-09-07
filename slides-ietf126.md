% Bundle-in-Bundle Encapsulation (BIBE)
% Rick Taylor (Aalyria Technologies)
% IETF 126 — DTN Working Group

# draft-taylor-dtn-bibe-00

- Bundle-in-Bundle Encapsulation: a bundle carried as the payload of one or more encapsulating bundles
- Replaces the expired draft-ietf-dtn-bibect-05 ("Replaces" relationship recorded on submission)
- This talk in two parts:
  - **Part 1** — why this document exists, and why I am holding the pen
  - **Part 2** — what is actually in it

<!-- Speaker context, not for slides: the bibect authorship substantially overlaps the CCSDS Orange Book authorship — the work did not stop, it moved venue. Every Part 1 argument must therefore read as an invitation back to the IETF, never as competition with CCSDS or criticism of the authors. The unanswerable point is change control: only the IETF can deprecate an RFC 9171 mechanism, so an IETF document must exist regardless of how good the Orange Book is. ADDED COMPLICATION: Rick is a WG chair — the real subtext is "chair uses the gavel to grab a parked document." Hat declaration up front, co-chair owns every process decision on this document (the consensus judgment on re-scope/rename, and the chair approval for the ietf- namespace resubmission), recusal stated explicitly and early. The pen-is-available offer matters double here. -->

# Part 1: Why This Document, and Why Now

- First, the hat: I am presenting as an individual contributor and proposed editor — **not as chair**
  - I have no chair role on this document: my co-chair runs every process question it raises, starting today
- The question hanging in the room: why is this person holding the pen on this work?
- Fair question. Four answers:
  1. Fragmentation must be deprecated, and deprecation needs a replacement document
  2. bibect expired — twice — and the custody-transfer coupling is why
  3. bibect is a WG document, and the WG owns its own work
  4. The discussion is continuing elsewhere — and it should be happening where this WG can see it

# 1. The Forcing Function: Fragmentation Must Go

- RFC 9171 ADU fragmentation is broken in practice:
  - Incompatible with BPSec deployments; prohibited for bundles bearing certain security blocks
  - Fragmentation and security are, in effect, mutually exclusive
- The direction of travel — here and at CCSDS — is to deprecate ADU fragmentation
- Process reality: you cannot deprecate a mechanism of an IETF Standards Track RFC without an IETF Standards Track replacement to point at
  - A CCSDS book cannot deprecate a feature of RFC 9171 — the replacement must live *here*
- No live IETF document → no deprecation. That is the hole this draft fills.

# 2. What Happened to bibect

- Five revisions over seven years (2018–2025), a four-year gap in the middle, expired again September 2025 — without completing
- It coupled encapsulation with a custody-transfer / acknowledgement mechanism that never found consensus in this room
- Encapsulation itself was never the controversial part — the blocker was the scope, not the authors
- Two lifecycles, same coupling, same outcome: this document changes the variable that failed, and keeps everything that didn't

# 3. Whose Work Is It? (a) Ownership

- bibect is a DTN WG document: the WG adopted this work in 2018
- WG documents belong to the WG, not to individuals — that is what adoption *means*
- Expiry is an administrative state, not a change of ownership: the work item never left this WG
- The WG may re-scope, rename, re-staff, and finish its own work, at any time, by its own consensus
- That is all that is being proposed here

# 3. Whose Work Is It? (b) Credit

- Nothing is being taken from anyone; the draft says so in as many words:
  - "The BIBE architecture is his" — Scott Burleigh, named Contributor
  - Montilla, Deaton, Caini — named Contributors with per-person contribution statements
  - "The editor claims no design novelty; the contribution of this document is to restate that body of work as a single self-contained specification"
  - Full lineage acknowledged: 2009 MITRE encapsulation draft → 2013 draft-irtf-burleigh-bibe → bibect → CCSDS SIS-DTN
- Editor is a role, not a reward: someone had to hold a pen to force the question
  - The pen is transferable, today, to anyone the WG prefers
  - Co-editors welcome; an original author stepping back in would be better still

# 4. The CCSDS Question

- CCSDS SIS-DTN is doing good work on exactly this problem — the segmentation mechanism in this draft *is* their design (prototyped in DTNME at NASA)
- This draft is not in competition with that work: it adopts the CCSDS representation deliberately, so the two strands stay aligned — and the draft credits the people who carried the work in both bodies
- But CCSDS documents are private until complete: this WG cannot see, review, or influence the discussion while it is happening
- The WG should have access to that discussion **as it happens** — in the open
- This is not a negotiation between strangers: several people in this room sit in both bodies
- And the change-control point is not optional: BP is an IETF protocol. However good the CCSDS document is, deprecating RFC 9171 fragmentation requires an IETF document — the two are complements, not rivals
- One open document lets IETF and CCSDS converge on a single mechanism, instead of discovering a fork after both have published

# The Proposal: An Update and a Rename — Not an Adoption Call

- This is **not** a request for adoption: the work is already adopted, and has been since 2018
- draft-taylor-dtn-bibe-00 is proposed as the next revision of the WG's existing work item, under a name matching its corrected scope
- The taylor- name is a staging artifact, nothing more: an individual cannot post into the ietf- namespace without chair action — and for this document, that chair action is my co-chair's alone
- On WG agreement, it is resubmitted as **draft-ietf-dtn-bibe-00**, with "Replaces: draft-ietf-dtn-bibect" recorded — datatracker continuity preserved
- What the room is being asked to confirm is exactly two things:
  - The re-scope: encapsulation only, custody transfer out
  - The rename that follows from it: bibect minus "ct" = bibe
- Whether the room agrees, and whether this counts as continuation rather than new work, is a consensus judgment — one I am recused from making
- And if the room reads it as new work instead: fine — that is an ordinary adoption call, run by my co-chair. Either road goes through WG consensus

# Part 1 Close

- To my co-chair: this document is yours to run — the re-scope/rename question, the consensus call, the namespace approval; I take no part in any of them
- Original authors: confirm you are content with Contributor listing — or take a bigger role
- Then let us argue about the technical content, not the provenance
- → Part 2

# Part 2: The Document at a Glance

- **Encapsulation only** — no custody transfer, no acknowledgement, no retransmission
  - Reliability belongs to convergence layers below, or custody / application signalling above — where CCSDS/ESA's Compressed Bundle Status Reporting (CBSR) work is already heading
  - The tunnel is deliberately transparent to anything layered above it, so the two compose
- **No administrative record, no IANA actions**
  - Encapsulation payload is ordinary application data at policy-configured decapsulation endpoints
  - No admin record type, no well-known service number — and no predictable target for unsolicited traffic
- **Segmentation included** — the CCSDS design, using the well-known transfer/segment pattern (as in TCPCLv4, BTPU — similar, but its own thing)
- **Specified as observable behaviour** — encapsulation function + decapsulation element; occupies the CLA position but is not required to be a CLA

# Use Cases

- **Security encapsulation:** BCB on the encapsulating payload encrypts the *entire* inner bundle, primary block included — traffic-analysis defence BPSec alone cannot provide
- **Cross-domain / service-provider:** domain policy applied to the encapsulating bundle; inner bundle untouched
- **Version encapsulation:** BPv6 ↔ BPv7, no discriminator needed — first octet disambiguates
- **Segmentation:** BPSec-compatible alternative to bundle fragmentation for size-limited paths
- **Traffic differentiation:** multiple decapsulation endpoints as flow identifiers, visible even when the payload is encrypted

# The IP Tunneling Analogy

- Stateless like GRE / IP-in-IP: no session, no negotiation — tunnel defined entirely by configuration at its two endpoints
- BCB on the encapsulating payload = IPsec tunnel mode; BPSec direct = transport mode
- Segmentation = "outer fragmentation" at the tunnel endpoints, transparent to the inner bundle
- Per-class decapsulation endpoints = the GRE Key field
- The hazards carry over too:
  - Recursive encapsulation loops — protection REQUIRED (no hop limit survives encapsulation)
  - RFC 6169 tunnel-security catalogue, translated to the BP layer

# Segmentation Mechanics

- The well-known transfer/segment pattern — TCPCLv4 has it, BTPU has it — reduced to the minimum, because BPv7 already provides the framing: each payload arrives whole or not at all, so only segment identification and reassembly rules are needed
- Wire format: CBOR array — size 1 = whole bundle; size 4 = `[transfer-id, total-length, segment-offset, data]`
  - Informative CDDL in the appendix; text governs
- Transfer ID scoped to (encapsulating node, decapsulation endpoint); no ordering semantics
- No windowing machinery — bundle expiry is the temporal bound; all bundles of a transfer share one expiry via a rolling lifetime calculation
- Segmented transfers require a singleton decapsulation endpoint
- Reassembly is OPTIONAL; unsupported segments are discarded

# Deliberate Non-Features

- Unreliable: no BIBE-layer acknowledgement or retransmission
- Fails silently: no post-delivery disposition signal — that would be the acknowledgement layer this protocol intentionally does not have
- Diagnostics need no new machinery: existing status reports observe every stage of the chain; delivery reports localise loss chunk by chunk
- Decapsulating node is a policy enforcement point: full inbound admission policy, no trust inherited from the tunnel

# Running Code

- This is a well-trodden concept — the predecessor formats have real implementations:
  - **ION** — the bibect reference implementation (admin-record format, custody included)
  - **µD3TN** — encapsulation only
  - **DTNME** — the CCSDS segmentation prototype, encapsulation only
- Two of the three, independently, implemented encapsulation and skipped the custody machinery — the scope of this document is where implementers already landed
- And of *this* document's format: Aalyria has a proof of concept of the segmentation mechanism running today
- A formal Implementation Status section (RFC 7942) belongs in a later revision, once there are implementations of this exact wire format to list

# Thank You

- **Draft:** [draft-taylor-dtn-bibe](https://datatracker.ietf.org/doc/draft-taylor-dtn-bibe/)
- **Repository:** github.com/ricktaylor/bibe
- **Contact:** rtaylor@aalyria.com
