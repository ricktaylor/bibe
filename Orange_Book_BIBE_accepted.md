<img src="media/image1.emf" style="width:4.83333in;height:0.83333in" />

Draft Recommendation for  
Space Data System Standards

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: center;"><blockquote>
<p><strong>BUNDLE IN BUNDLE ENCAPSULATION (BIBE)</strong></p>
</blockquote></td>
</tr>
</tbody>
</table>

Draft Experimental Specification

CCSDS 000.0-W-0

White Book

July 2026

AUTHORITY

|     |           |                                           |     |
|-----|-----------|-------------------------------------------|-----|
|     |           |                                           |     |
|     | Issue:    | Draft Experimental Specification, Issue 0 |     |
|     | Date:     | July 2026                                 |     |
|     | Location: | Not Applicable                            |     |
|     |           |                                           |     |

**(WHEN THIS EXPERIMENTAL SPECIFICATION IS FINALIZED, IT WILL CONTAIN THE FOLLOWING STATEMENT OF AUTHORITY:)**

This document has been approved for publication by the Consultative Committee for Space Data Systems (CCSDS). The procedure for review and authorization of CCSDS documents is detailed in *Organization and Processes for the Consultative Committee for Space Data Systems* (CCSDS A02.1-Y-4).

This document is published and maintained by:

CCSDS Secretariat

National Aeronautics and Space Administration

Washington, DC, USA

Email: secretariat@mailman.ccsds.org

FOREWORD

Attention is drawn to the possibility that some of the elements of this document may be the subject of patent rights. CCSDS shall not be held responsible for identifying any or all such patent rights.

Through the process of normal evolution, it is expected that expansion, deletion, or modification of this document may occur. This document is therefore subject to CCSDS document management and change control procedures which are defined in *Organization and Processes for the Consultative Committee for Space Data Systems* (CCSDS A02.1-Y-4). Current versions of CCSDS documents are maintained at the CCSDS Web site:

http://www.ccsds.org/

Questions relating to the contents or status of this document should be addressed to the CCSDS Secretariat at the address indicated on page i.

At time of publication, the active Member and Observer Agencies of the CCSDS were:

<u>Member Agencies</u>

- Agenzia Spaziale Italiana (ASI)/Italy.

- Canadian Space Agency (CSA)/Canada.

- Centre National d’Etudes Spatiales (CNES)/France.

- China National Space Administration (CNSA)/People’s Republic of China.

- Deutsches Zentrum für Luft- und Raumfahrt (DLR)/Germany.

- European Space Agency (ESA)/Europe.

- Federal Space Agency (FSA)/Russian Federation.

- Instituto Nacional de Pesquisas Espaciais (INPE)/Brazil.

- Japan Aerospace Exploration Agency (JAXA)/Japan.

- National Aeronautics and Space Administration (NASA)/USA.

- UK Space Agency/United Kingdom.

<u>Observer Agencies</u>

- Austrian Space Agency (ASA)/Austria.

- Belgian Science Policy Office (BELSPO)/Belgium.

- Central Research Institute of Machine Building (TsNIIMash)/Russian Federation.

- China Satellite Launch and Tracking Control General, Beijing Institute of Tracking and Telecommunications Technology (CLTC/BITTT)/China.

- Chinese Academy of Sciences (CAS)/China.

- China Academy of Space Technology (CAST)/China.

- Commonwealth Scientific and Industrial Research Organization (CSIRO)/Australia.

- Danish National Space Center (DNSC)/Denmark.

- Departamento de Ciência e Tecnologia Aeroespacial (DCTA)/Brazil.

- Electronics and Telecommunications Research Institute (ETRI)/Korea.

- European Organization for the Exploitation of Meteorological Satellites (EUMETSAT)/Europe.

- European Telecommunications Satellite Organization (EUTELSAT)/Europe.

- Geo-Informatics and Space Technology Development Agency (GISTDA)/Thailand.

- Hellenic National Space Committee (HNSC)/Greece.

- Hellenic Space Agency (HSA)/Greece.

- Indian Space Research Organization (ISRO)/India.

- Institute of Space Research (IKI)/Russian Federation.

- Korea Aerospace Research Institute (KARI)/Korea.

- Ministry of Communications (MOC)/Israel.

- Mohammed Bin Rashid Space Centre (MBRSC)/United Arab Emirates.

- National Institute of Information and Communications Technology (NICT)/Japan.

- National Oceanic and Atmospheric Administration (NOAA)/USA.

- National Space Agency of the Republic of Kazakhstan (NSARK)/Kazakhstan.

- National Space Organization (NSPO)/Chinese Taipei.

- Naval Center for Space Technology (NCST)/USA.

- Netherlands Space Office (NSO)/The Netherlands.

- Research Institute for Particle & Nuclear Physics (KFKI)/Hungary.

- Scientific and Technological Research Council of Turkey (TUBITAK)/Turkey.

- South African National Space Agency (SANSA)/Republic of South Africa.

- Space and Upper Atmosphere Research Commission (SUPARCO)/Pakistan.

- Swedish Space Corporation (SSC)/Sweden.

- Swiss Space Office (SSO)/Switzerland.

- United States Geological Survey (USGS)/USA.

PREFACE

This document is a CCSDS Experimental Specification. Its Experimental status indicates that it is part of a research or development effort based on prospective requirements, and as such it is not considered a Standards Track document. Experimental Specifications are intended to demonstrate technical feasibility in anticipation of a ‘hard’ requirement that has not yet emerged. Experimental work may be rapidly transferred onto the Standards Track should a hard requirement emerge in the future.

<span id="_Toc231715152" class="anchor"></span>DOCUMENT CONTROL

|  |  |  |  |
|----|----|----|----|
| **Document** | **Title and Issue** | **Date** | **Status** |
| CCSDS 000.0-W-0 | Bundle In Bundle Encapsulation (BIBE), Proposed Draft Experimental Specification, Issue 0 | May 2026 |  |

<span id="_Toc231715153" class="anchor"></span>CONTENTS

Section Page

[DOCUMENT CONTROL [v](#_Toc231715152)](#_Toc231715152)

[CONTENTS [vi](#_Toc231715153)](#_Toc231715153)

[1 Introduction [1-1](#introduction)](#introduction)

[1.1 Purpose [1-1](#purpose)](#purpose)

[1.2 scope [1-1](#scope)](#scope)

[1.3 applicability [1-1](#applicability)](#applicability)

[1.4 document structure [1-2](#document-structure)](#document-structure)

[1.5 conventions and Definitions [1-2](#conventions-and-definitions)](#conventions-and-definitions)

[1.6 patented technologies [1-3](#patented-technologies)](#patented-technologies)

[1.7 References [1-4](#references)](#references)

[2 Overview [2–1](#overview)](#overview)

[2.1 Architecture [2–1](#architecture)](#architecture)

[2.2 summary of functions [2–3](#summary-of-functions)](#summary-of-functions)

[3 BUNDLE In Bundle Encapsulation Protocol CONSIDERATIONS [3–1](#bundle-in-bundle-encapsulation-protocol-considerations)](#bundle-in-bundle-encapsulation-protocol-considerations)

[3.1 OVERVIEW [3–1](#overview-1)](#overview-1)

[3.2 BUNDLE IN BunDLE Encapsulation (BIBE) CLAs [3–1](#bundle-in-bundle-encapsulation-bibe-clas)](#bundle-in-bundle-encapsulation-bibe-clas)

[3.3 BIBE Protocol data units [3–1](#bibe-protocol-data-units)](#bibe-protocol-data-units)

[3.4 BPDU Transmission [3–2](#bpdu-transmission)](#bpdu-transmission)

[3.5 BPDU Reception [3–4](#bpdu-reception)](#bpdu-reception)

[3.6 BPDU FORMAT [3–4](#bpdu-format)](#bpdu-format)

| <u>Figure</u> |     |
|---------------|-----|

[1‑1 Bit Numbering Convention [1-3](#_bookmark9)](file:///C:/Users/jwjacks4/AppData/Local/Microsoft/Windows/INetCache/Content.Outlook/I63MIT4Q/IMC_Orange_Book_Draft.docx#_Toc147141195)

2‑1 BIBE Bundle Transmission Example [2-](#_Toc147141197)1

# Introduction

## Purpose

The purpose of this Experimental Specification is to define procedures for bundle in bundle encapsulation with bundle protocol version 7(reference \[3\]) across delay tolerant networks. Bundle in bundle encapsulation can be used for several different purposes including service provider encapsulation, bundle segmentation, and version encapsulation to allow BPv6 to operate across BPv7 networks.

## scope

This Experimental Specification defines:

1)  BIBE Convergence Layer Adapters;

2)  BIBE Protocol Data Units;

3)  BPDU Transmission;

4)  BPDU Reception;

## applicability

This Experimental Specification may be applied to future data communications over delay tolerant networking between CCSDS Agencies in cross-support situations. It includes comprehensive specifications of the data formats and procedures for interagency cross support. It is neither a specification of, nor a design for, real systems that may be implemented for existing or future missions.

The Experimental Specification described in this document may be invoked through the normal standards program of each CCSDS Agency and is applicable to those missions for which cross support based on capabilities described in this Experimental Specification is anticipated. Where mandatory capabilities are clearly indicated in sections of this Experimental Specification, they must be implemented when this document is used as a basis for cross support. Where options are allowed or implied, implementation of these options is subject to specific bilateral cross support agreements between the Agencies involved.

### Rationale

This Experimental Specification facilitates the definition of a special encapsulation method to achieve scenarios that would otherwise be impossible or difficult to do. Numerous use cases exist for BIBE, some of which include security encapsulation, allowing the encapsulation of a bundle so that BPSec can be used to encrypt the entirety of the internal bundle, segmentation, allowing the segmentation of larger bundles over Convergence Layer Adapters for protocols which do not support segmentation, and edge node encapsulation, allowing for network providers to encapsulate bundles entering their network so they can apply provider specific services.

The CCSDS believes it is important to document the rationale underlying the recommendations chosen, so that future evaluations of proposed changes or improvements will not lose sight of previous decisions. Where appropriate, rationale has been included in the description.

## document structure

This document is divided into five numbered sections and five annexes.

1)  section 1 presents the purpose, scope, applicability, rationale, document structure, conventions, and references;

2)  section 2 provides an overview of the architecture and summary of functions of Bundle in Bundle Encapsulation;

3)  section 3 discusses BIBE protocol definitions;

4)  annex A defines the service provided to the users;

5)  annex B discusses prototype testing;

## conventions and Definitions 

### Nomenclature 

The following conventions apply throughout this specification:

1)  the words ‘shall’ and ‘must’ imply a binding and verifiable specification;

2)  the word ‘should’ implies an optional, but desirable, specification;

3)  the word ‘may’ implies an optional specification;

4)  the words ‘is’, ‘are’, and ‘will’ imply statements of fact.

NOTE – These conventions do not imply constraints on diction in text that is clearly informative in nature.

### conventions

In this document, the following convention is used to identify each bit in an *N*-bit field. The first bit in the field to be transmitted (i.e., the most left-justified when drawing a figure) is defined to be ‘Bit 0’, the following bit is defined to be ‘Bit 1’, and so on up to ‘Bit *N*−1’. When the field is used to express a binary value (such as a counter), the Most Significant Bit (MSB) shall be the first transmitted bit of the field, that is, ‘Bit 0’ (see figure[1-1](file:///C:/Users/jwjacks4/AppData/Local/Microsoft/Windows/INetCache/Content.Outlook/I63MIT4Q/IMC_Orange_Book_Draft.docx#_bookmark9))..

> Bit 0 Bit *N*−1
>
> *N*-Bit Data Field
>
> First Bit Transmitted = MSB

<span id="_bookmark9" class="anchor"></span>Figure 1‑1: Bit Numbering Convention

In accordance with standard data-communications practice, data fields are often grouped into 8- bit ‘words’, which conform to the above convention. Throughout this Specification, such an 8- bit word is called an ‘octet’.

The numbering for octets within a data structure starts with ‘0’.

## patented technologies

The CCSDS draws attention to the fact that it is claimed that compliance with this document may involve the use of patents.

The CCSDS takes no position concerning the evidence, validity, and scope of these patent rights.

The holders of these patent rights have assured the CCSDS that they are willing to negotiate licenses under reasonable and non-discriminatory terms and conditions with applicants throughout the world. In this respect, the statements of the holders of these patent rights are registered with CCSDS. Information can be obtained from the CCSDS Secretariat at the address indicated on page i. Contact information for the holders of these patent rights is provided in annex B.

Attention is drawn to the possibility that some of the elements of this document may be the subject of patent rights other than those identified above. The CCSDS shall not be held responsible for identifying any or all such patent rights.

## References

The following publications contain provisions which, through reference in this text, constitute provisions of this Experimental Specification. At the time of publication, the editions indicated were valid. All publications are subject to revision, and users of this Experimental Specification are encouraged to investigate the possibility of applying the most recent editions of the documents indicated below. The CCSDS Secretariat maintains a register of currently valid CCSDS publications.

\[1\] *CCSDS Publications Manual*. Issue 4. CCSDS Record (Yellow Book), CCSDS A20.0-Y-4. Washington, D.C.: CCSDS, April 2014.

\[2\] *Organization and Processes for the Consultative Committee for Space Data Systems*. Issue 4. CCSDS Record (Yellow Book), CCSDS A02.1-Y-4. Washington, D.C.: CCSDS, April 2014.

\[3\] Burleigh, S., Fall, K., Birrane, E., “Bundle Protocol Version 7” RFC9171. Reston, Virginia, ISOC, January 2022.

\[4\] CCSDS Bundle Protocol Specification. CCSDS Experimental Specification (Orange Book), CCSDS 734.20-O-1. CCSDS, April 2025.

\[Only references required for the implementation of the specification are listed in the References subsection. (See reference \[1\] for additional information on this subsection.)\]

# Overview 

## Architecture 

BIBE acts as a Convergence Layer Adapter (CLA), as defined in the specification for BPv7 (reference \[4\]). Like any CLA, a BIBE CLA (BCLA) comprises an output/sending function (a BIBE CLO) and an input/delivery function (a BIBE CLI). When a bundle protocol agent invokes the services of a BCLO to send a bundle:

- The BCLO generates a sequence of one or more BIBE protocol data units (BPDUs), each of which encapsulates part or all of that bundle (termed the “source bundle”).

  - A BPDU that contains an entire source bundle is termed a “simple” BPDU.

  - A BPDU that encapsulates only part of the source bundle (that is, a segment of the source bundle) is termed a “segment” BPDU.

- For each BPDU, the BCLO – acting in the capacity of a BP application – requests transmission of a bundle (at a second, underlying layer of bundle protocol support) whose payload is that BPDU.

  - Each such bundle is termed a “BIBE bundle.”

Upon reception of a BIBE bundle at its destination node, the node’s bundle protocol agent delivers the payload of the BIBE bundle – a BPDU – to the destination endpoint in the normal way. In order for BIBE operation to succeed, the application registered at that endpoint must be a BIBE CLI, acting in the capacity of a BP application. When a bundle protocol agent delivers a BPDU to such an endpoint:

- If the delivered BPDU is a simple BPDU, the BIBE CLI delivers the source bundle encapsulated in the BPDU to the bundle protocol agent (at a second, overlying layer of bundle protocol support) for forwarding and/or delivery in the standard manner.

- If the delivered BPDU is a segment BPDU, the BIBE CLI inserts the source bundle segment encapsulated in the BPDU into a reassembly structure for the indicated source bundle; when this results in completion of the reassembly of the source bundle, the BIBE CLI delivers the reassembled source bundle to the bundle protocol agent (again, at a second, overlying layer of bundle protocol support) for forwarding and/or delivery in the standard manner.

<figure>
<img src="media/image8.png" style="width:3.76059in;height:4.95506in" />
<figcaption><p>Figure 2-1 BIBE – BP Agent Architecture</p></figcaption>
</figure>

## summary of functions 

### general

This Experimental Specification describes the functions necessary for bundle in bundle encapsulation using BPv7. Figure 2-1 depicts a simple DTN nodal infrastructure with a user sending a single bundle, sending it through a service provider network where it gets encapsulated and then decapsulated upon exiting the service provider network, and finally the bundle being delivered to the user dtn node on the other end.

Service Provider  
DTN Node

Service Provider  
DTN Node

User

DTN Node

User

DTN Node

Figure 2‑2: BIBE Bundle Transmission Example

# BUNDLE In Bundle Encapsulation Protocol CONSIDERATIONS

## OVERVIEW

There are considerations relevant to appropriate usage of BIBE. These considerations provide the framework for ensuring the proper encapsulation and decapsulation of bundles.

## BUNDLE IN BunDLE Encapsulation (BIBE) CLAs

BIBE convergence-layer adapters (BCLAs) are implemented within the application-specific elements of the application agents (AA) of BP nodes that conform to the BIBE protocol specification. The node of which a given BCLA is one component is termed the BCLA's "local node". A BP node that includes a BCLA is termed a "BIBE node".

## BIBE Protocol data units

The BPDU encapsulated in the payload block of a BIBE bundle is represented as follows.

Each BPDU SHALL be represented as a CBOR array of size N.

When N is 1 the sole member of the array SHALL be a single BP bundle, termed the "source bundle" as noted earlier, represented as a CBOR byte string of definite length.

When N is 4, the first 3 members of the array shall be segmentation information as defined below and the fourth member of the array shall be a *segment* of the source bundle as described below, similarly represented as a CBOR byte string of definite length.

All other values of N are not defined here and are reserved for future use.

NOTE – A BIBE protocol data unit (BPDU) is not an administrative record; its destination shall not be an administrative endpoint.

## BPDU Transmission

For each destination endpoint to which BIBE bundles will be forwarded:

- The maximum number of source bundle bytes that may be encapsulated by this BCLA in the BPDU that is the payload for any single BIBE bundle forwarded to that endpoint – termed the endpoint’s “segmentation threshold” – shall be established by management.

- For use in the labeling of segment BPDUs, the BCLA shall maintain a transmission counter.

When a BCLA is requested by the bundle protocol agent to send a bundle to the peer BCLA(s) included in the destination BP endpoint identified by a specified BP endpoint ID:

- If the size of the bundle does not exceed the segmentation threshold for that endpoint, then the BCLA shall generate a simple BPDU encapsulating the entire bundle. The sole member of that BPDU’s CBOR array shall be the entire bundle formatted as a CBOR byte string of definite length. The BCLA shall request transmission of a bundle, destined for the indicated endpoint, whose payload is that BPDU.

- Otherwise the bundle shall be segmented as follows:

  - The value of the transmission counter for the indicated endpoint shall be increased by 1, and the result shall be used as the transfer ID for all BIBE bundles generated from this source bundle.

  - Given bundle length L and segmentation threshold T, the number Q of segment BPDUs to be generated shall be given by L/T, plus 1 if L/T leaves a remainder.

  - For each generated segment BPDU *n*, where *n* varies from 1 to Q, the 4<sup>th</sup> member of the BPDU’s CBOR array shall contain a *segment* of the source bundle comprising bytes A through B where:

    - A is computed as ((n – 1) \* T) + 1.

    - B is computed as (A + T) – 1 or L, whichever is less.

  - The BCLA shall generate these BPDUs, formatted as described in 3.4.1 below, and shall request transmission of bundles (destined for the indicated endpoint) whose payloads are these BPDUs.

All octets of the source bundle SHALL be encapsulated within the output BPDU(s) of the CLO, in the order in which they appear in the source bundle, without alteration or duplication.

Selection of the values of the parameters governing the forwarding of BIBE bundle(s), other than the destination endpoint ID, is an implementation matter. The parameter values governing the forwarding of the BPDU's encapsulated bundle MAY be consulted for this purpose.

Note that BIBE bundles, like any others, are subject to forwarding and acquisition policy established by node management. In particular, note that the BPDUs generated by a BCLA are plain text; any encryption of BPDUs and their encapsulated source bundles or source bundle segments will be performed not by the BCLA but by the bundle protocol agent in the course of generating the BIBE bundles. Similarly, decryption of encrypted BPDUs and their encapsulated source bundle material will be performed not by the BCLA but by the bundle protocol agent in the course of receiving BIBE bundles and delivering their payloads.

NOTE – IETF RFC9171 \[3\] is used as BPv7 reference here as the CCSDS Bundle Protocol Specification \[4\] refers to RFC9171 for detailed behaviors.

### BPDU SEGMENTATION

NOTE – BIBE Segmentation is an optional capability. It may be necessary for interoperability purposes, supporting forwarding or large(r) bundles on nodes using non-segmenting Convergence Layer Adapters (such as CCSDS Encapsulation Packet Protocol - EPP). It has been introduced to cope with the fragmentation challenge in BPv7 (fragmentation in BPv7 has been deemed incompatible with BPSec).

The elements of a segment BPDU’s CBOR array shall be as follows:

1.  Transfer ID, obtained as described above, formatted as a CBOR unsigned integer.

2.  The total length L of the source bundle as noted above, formatted as a CBOR unsigned integer.

3.  Segment offset, the applicable value of ((*n* – 1) \* T) as noted above, formatted as a CBOR unsigned integer.

4.  The encapsulated segment of the source bundle as noted above, formatted as a CBOR byte string of definite length.

Transfer-id shall be unique for a given source during the lifetime of the corresponding segmented BIBE Bundle

NOTE – When performing segmentation, Time-to-Live (TTL) of the encapsulating bundle may be different than the TTL of the encapsulated bundle segments. Implementations should have this into consideration.

## BPDU Reception

When a BCLA receives a BPDU from the bundle protocol agent (that is, upon delivery of the payload of a BIBE bundle):

- If the BPDU is a simple BPDU (as indicated by an array size value of 1), then the encapsulated source bundle SHALL be delivered from the BCLA to the bundle protocol agent.

- If the BPDU is a segment BPDU (as indicated by an array size value of 4) and segmentation is supported by the receiving BCLA:

  - If the source bundle offset of any byte of the encapsulated source bundle segment exceeds the length of the source bundle, the BCLA SHALL discard the segment and, if so requested, issue a “Deleted” bundle status report with reason code “Failed BIBE reassembly.”

  - Otherwise:

    - The BLCA SHALL insert the encapsulated source bundle segment into a source bundle reassembly structure, the nature of which shall be an implementation matter.

    - If insertion of this segment results in the completion of reassembly of the source bundle, then the reassembled source bundle shall be delivered to the bundle protocol agent.

  - 

  - 

- If the BPDU is a segment BPDU and segmentation is not supported by the receiving BCLA, then the BCLA SHALL discard the encapsulated source bundle segment and, if so requested, issue a “Deleted” bundle status report with Reason Code “BIBE segmentation unsupported”.

Upon delivery of a source bundle from the BCLA to the bundle protocol agent, reception of the source bundle SHALL be performed as defined in section 5.6 of the Bundle Protocol specification in the usual manner: the source bundle may be forwarded, delivered, etc.

## BPDU FORMAT

BIBE-PDU = definite-length CBOR array \[

transfer-id: unsigned CBOR int,

total-length: unsigned CBOR int,

segmented-offset: unsigned CBOR int,

encapsulated-bundle-segment: definite-length CBOR byte string,

\]

NOTE – For single bundles without fragmentation, transfer-id, total-length and segmented-offset are omitted.

########   SECURITY, SANA, AND PATENT CONSIDERATIONS  infORMATIVE

\[Annexes contain ancillary information. Normative annexes precede informative annexes. Informative references are placed in an informative annex. (See reference \[1\] for discussion of the kinds of material contained in annexes.)\]

**A1 SECURITY CONSIDERATIONS**

As BIBE is an enhancement to the bundle protocol, security considerations follow closely to those referenced in bundle protocol itself. Security considerations for BPv7 are handled by BPSec (reference \[3\]).

While general security considerations follow closely to BPv7 an interesting use case arises with BIBE. For standard bundles to be routable while using BPSec, parts of the bundle must be left unencrypted. With BIBE the entire source bundle becomes the payload of the BIBE bundle which then allows for encryption of the entire source bundle.

A potential DoS attack can be generated by transmitting incomplete segmented bundles with a very large Time-to-Live. Considerations must be taken on implementations to avoid bundle storage depletion. Approaches including authentication and policy help (such as TTL overrides) reducing this attack vector.

**A2 SANA CONSIDERATIONS**

\[See CCSDS 313.0-Y-1, *Space Assigned Numbers Authority (SANA)—Role, Responsibilities, Policies, and Procedures* (Yellow Book, Issue 1, July 2011).\]

To facilitate interoperability, a new well-known service number is required for BIBE. Note this does not preclude BIBE operating in other service number(s). Using a SANA-reserved number facilitates interoperability, as IANA already recognizes the SANA reserved pool of service numbers.

**A3 PATENT CONSIDERATIONS**

\[See CCSDS A20.0-Y-4, *CCSDS Publications Manual* (Yellow Book, Issue 4, April 2014).\]

########   Prototype Testing  (Informative)

NASA has developed a prototype of the specification as part of the DTNME implementation. For the testing setup a four-node configuration using the version encapsulation use case was used as noted in Figure C-1.

BPv7

Node C

BPv7

Node B

BPv6

Node A

BPv6

Node D

**Figure C-1: Prototype Test Setup**

The application used for this test was ping_me along with its counterpart echo_me. Node A was assigned ipn:10, Node B was assigned ipn:20, Node C was assigned ipn:30, and Node D was assigned ipn:40. Node A generated BPv6 bundles and forwarded them to Node B. Node B included a forwarding rule to encapsulate the outgoing bundles headed to Node C using BIBE. The encapsulated bundles were created and per the specification and forwarded to Node C where they were decapsulated. Node C then forwarded the decapsulated bundles to Node D where it was delivered to the application. Upon the delivery of the ping message to echo_me a reply was generated and sent across the reverse path. Where once again the BPv6 bundle was encapsulated using BIBE to transit between Node C and Node B.
