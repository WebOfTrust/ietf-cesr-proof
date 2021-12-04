---
title: "CESR Proof Signatures"
abbrev: "CESR-PROOF"
docname: draft-pfeairheller-cesr-proof-latest
category: info

ipr: trust200902
area: TODO
workgroup: TODO Working Group
keyword: Internet-Draft

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
  -
    name: Phil Feairheller
    organization: GLEIF
    email: Philip.Feairheller@gleif.org

normative:
  CESR:
    target: https://datatracker.ietf.org/doc/draft-ssmith-cesr/
    title: Composable Event Streaming Representation (CESR)
    author:
      ins: S. Smith
      name: Samuel M. Smith
      org: ProSapien LLC
    date: 2021

  SAID:
    target: https://datatracker.ietf.org/doc/draft-ssmith-said/
    title: Self-Addressing IDentifier (SAID)
    author:
      ins: S. Smith
      name: Samuel M. Smith
      org: ProSapien LLC
    date: 2021

informative:
  KERI:
    target: https://arxiv.org/abs/1907.02143
    title: Key Event Receipt Infrastructure (KERI)
    author:
      ins: S. Smith
      name: Samuel M. Smith
      org: ProSapien LLC
    date: 2021

  JSON:
    target: https://www.json.org/json-en.html
    title: JavaScript Object Notation Delimeters

  JSONPath:
    target: https://tools.ietf.org/id/draft-goessner-dispatch-jsonpath-00.html
    title: JSONPath -- XPath for JSON

tags: IETF, CESR, SAID, KERI, ACDC

--- abstract

CESR Proof Signatures are an extension to the Composable Event Streaming Representation (CESR) that provide transposable
cryptographic signature attachments on self-addressing data (SAD). A SAD (such as an Authentic Chained Data Container
(ACDC) Verifiable Credential) may be signed with a CESR Proof Signature and streamed alongside any other CESR content.
In addition, a signed SAD can be embedded inside another SAD and the CESR proof signature attachment can be transposed
to the outer SAD and streamed without losing any cryptographic integrity.

--- middle

# Introduction

Composable Event Streaming Representation (CESR) is a dual text-binary encoding format that has the unique property of
text-binary concatenation composability. The CESR specification not only provides the definition of the streaming format
but also the codes needed for creating and sharing all event types and signatures on all event types for the Key Event
Receipt Infrastructure (KERI). While all KERI event messages are self-addressing data (SAD), there is a broad class of
SADs that are not KERI events but that require signature attachments. ACDC Verifiable credentials fit into this class of
SADs. With more complex data structures represented as SADs, such as verifiable credentials, there is a need to provide
signature attachments on nested subsets of SADs. Similar to indices in indexed controller signatures in KERI that
specify the location of the public key they represent, nested SAD signatures need a path mechanism to specify the exact
location of the nested content that they are signing. CESR Proof Signatures provide this mechanism with the CESR SAD
Path Language and a new attachment code, detailed below.

## Transposable Signature Attachments

There are several events in KERI that can contain context specific embedded self-addressing data (SADs). Exchange
events (`exn`) for peer-to-peer communication and Replay events (`rpy`) for responding to data requests as well as
Expose events (`exp`) for providing anchored data are all examples of KERI events that contain embedded SADs as part of
their payload. If the SAD payload for one of these event types is signed with a CESR attachment, the resulting structure
is not embeddable in one of the mapping serializations (JSON, CBOR, MessagePack) supported by CESR. To solve this
problem, CESR Proof Signatures are transposable in that an entire signature group on any given SAD can be transposed to
attach to the end of an outer SAD without losing its meaning. This unique feature is provided by a root path designation
in the outermost attachment code of any SAD signature group that can be updated to point to the embedding location of
the outer SAD. Protocols for verifiable credential issuance and proof presentation can be defined using this capability
to embed the same verifiable credential SAD at and location in an outer `exn` message as appropriate for the protocol
without having to define a unique signature scheme for each protocol.

## Selective Disclosure

## One Time Use Credentials

# CESR SAD Path Language

CESR Proof Signatures defines a SAD Path Language to be used in signature attachments for specifying the location of the
SAD content within the signed SAD that a signature attachment is verifying. This path language has a more limited scope
than alternatives like JSONPath and is therefore simpler and easier to encode in CESR signature attachments. SADs in
CESR and therefore CESR Proof Signatures require static field ordering of all maps. The SAD path language takes
advantage of this feature to allow for a base64 compatible syntax into SADs even when a SAD uses non-base64 compatible
characters for field labels.

## Description and Usage

The SAD path language contains a single reserved character, the `-` (dash) character. Similar to the `/` (forward slack)
character in URLs, the `-` in the SAD Path Language is the path separator between components of the path. The `-` was
selected because it is a one of the valid base64 characters.

The simplest path in the SAD Path Language is a single `-` character representing the root path which specifies the top
level of the SAD content.

Root Path
``` - ```

After the root path, path components follow, delimited by the `-` character. Path components may be integer indices into
field labels or arrays or may be full field labels. No wildcards are supported by the SAD Path Language

An example SAD Path using only labels that resolve to map contexts follows:

```
-a-personal
```

In addition, integers can be specified and their meaning is dependent on the context of the SAD.

```
-1-12-personal-0
```

The rules for a SAD Path Language processor are simple. If a path consists of only a single `-`, it represents the root
of the SAD and therefore the entire SAD content. Following any `-` character is a path component that points to a field
if the current context is a map in the SAD or is an index of an element if the current context is an array. It is an
error for any sub-path to resolve to a value this is not a map or an array.

The root context (after the initial `-`) is always a map. Therefore, the first path component represents a field of that
map. The SAD is traversed following the path components as field labels or indexes in arrays until the end of the path
is reached. The value at the end of the path is then returned as the resolution of the SAD Path. If the current context
is a map and the path component is an integer, the path component represents an index into fields of the map. This
feature takes advantage of the static field ordering of SADs and is used against any SAD that contains field labels that
use non-base64 compatible characters or the `-` character. Any combination of integer and field label path components
can be used when the current context is a map. All path components MUST be an integer when the current context is an
array.

## SAD Path Examples

This section provides some more examples for SAD Path expressions. The examples are based on Authentic Chained Data
Containers (ACDCs) representing verifiable credentials.

```json
{
  "v": "ACDC10JSON00011c_",
  "d": "EBdXt3gIXOf2BBWNHdSXCJnFJL5OuQPyM5K0neuniccM",
  "i": "did:keri:EmkPreYpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPM",
  "s": "E46jrVPTzlSkUPqGGeIZ8a8FWS7a6s4reAXRZOkogZ2A",
  "a": {
    "d": "EgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PY",
    "i": "did:keri:EQzFVaMasUf4cZZBKA0pUbRc9T8yUXRFLyM1JDASYqAA",
    "dt": "2021-06-09T17:35:54.169967+00:00",
    "ri": "did:keri:EymRy7xMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggt",
    "LEI": "254900OPPU84GM83MG36",
    "personal": {
      "legalName": "John Doe",
      "home-city": "Durham"
    }
  },
  "p": [
    {
      "qualifiedIssuerCredential": {
        "d": "EIl3MORH3dCdoFOLe71iheqcywJcnjtJtQIYPvAu6DZA",
        "i": "Et2DOOu4ivLsjpv89vgv6auPntSLx4CvOhGUxMhxPS24"
      }
    },
    {
      "certifiedLender": {
        "d": "EglG9JLG6UhkLrrv012NPuLEc1F3ne5vPH_sHGP_QPN0",
        "i": "E8YrUcVIqrMtDJHMHDde7LHsrBOpvN38PLKe_JCDzVrA"
      }
    }
  ]
}
```

Figure 1. Example ACDC Credential SAD

The examples in Table 1 represent all the features of the SAD Path language when referring to the SAD in Figure 1.

|   SAD Path   | Result                            | Notes |
|:-------------|:----------------------------------|:------|
|  -           | The root of the SAD               | Root Path |
|  -a-personal | The personal map of the a field   | Only field labels |
|  -4-5        | The personal map of the a field   | Only field indices |
|  -4-5-legalName        | "John Doe"   | Mixing field labels and indices |
|  -a-personal-1        | "Durham"   | Required to use field index because field label contains `-`  |
|  -p-1        | The second element in the p array | Array context and index |
| -p-0-0-d     | "EIl3MORH3dCdoFOLe71iheqcywJcnjtJtQIYPvAu6DZA" | Mixing field labels, field indices and array indices |
| -p-0-certifiedLender-i | "E8YrUcVIqrMtDJHMHDde7LHsrBOpvN38PLKe_JCDzVrA" |  |

## CESR Encoding for SAD Path Language

# CESR Attachments

The specification adds 2 *Counter Four Character Codes* to the Master Code Table and 1 *Small Variable Raw Size Code
Type*
to the Master Code Table (which results in 3 new code entries):

|   Code   | Description                                                                         | Code Length | Count or Index Length | Total Length |
|:--------:|:----------------------------------|:------------:|:-------------:|:------------:|
|          |                        **Counter Four Character
Codes**                          |             |              |              |
|   -K##   | Count of attached qualified Base64 indexed controller signatures                    |      4      |       2      |       4      |
|   -J##   | Count of attached qualified Base64 indexed witness signatures                       |      4      |       2      |       4      |
|          |                        **Small Variable Raw Size
Code**                          |             |              |              |
|   4A##   | Size of SAD Path with 0 Lead Bytes                                                  |      4      |       2      |       4      |
|   5A##   | Said of SAD Path with 1 Lead Byte                                                   |      4      |       2      |       4      |
|   6A##   | Size of SAD Path with 2 Lead Bytes                                                  |      4      |       2      |       4      |

## Additional Count Codes

## Streamable SADs

## Nested Partial Signatures

## Transposable Signature Attachments

# CESR Signatures

Introduction to CESR Signatures

## Signatures with Non-Transferable Identifiers

## Signatures with Transferable Identifiers

## Selective Disclosure

### Small Variable Raw Size Path Code

# CESR Proof Signature Attachments

## -J

```
-J 1 (Path, TransIdxSigGroup)
```

## -K

```
-K Count of Sigs (Root Path, Path Sig Groups)
```

## Embedding and Transposable Signatures

It is required to embed this credential in any other KERI message (eg. `rpy`, `exp`, `exn`) along with the CESR Proof.
Since the proof is a CESR attachment, this message cannot simply be embedded in another JSON document. Therefore, a
method is needed to apply the attachment to the outermost message while maintaining its location pointers and meaning
for the nested SAD that it represents.

Given the following signed SAD:

```
{
  "v": "ACDC10JSON00011c_",
  "d": "EpcEvrX2gGTpmKbIG25GSA7_LsWwwzVQ6aUilgBubpGI",
  "i": "EmSIYYxvgtKn9jAp8GcK3fXOwTeyBIcAnRnyrLNfKjVI",
  "s": "EZdaE1HCu2ZhyIhpXTWfGSLS2kirKexaC-4up3sIUz1I",
  "a": {
    "d": "EG7PfBZbyXS0huxbnvMD-jOdTnDCCaT4CO6xId-ATNWg",
    "i": "EhwC_-ISX9helBDUSKU5j_VEU3G0Jp6ZFRD9dTb4-5c0",
    "dt": "2021-06-09T17:35:54.169967+00:00",
    "ri": "EymRy7xMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggt",
    "LEI": "254900YH3ZCDPE1E5306",
    "personal": {
      "d": "ExSxzzsvGlhO6W0O1_-k5Api8NgD8dDHLgazBxTqi_z4",
      "n": "Q8rNaKITBLLA96Euh5M5v4o3fRl1Bc54xdM-bOIHUjY",
      "personLegalName": "Anne Jones",
      "engagementContextRole": "Project Manager"
    }
  }
}-KAC  -  -JAB  -  -FABELfzj-TkiKYWsNKk2WE8F8VEgbu3P-_HComVHcKrvGmY0AAAAAAAAAAAAAAAAAAAAAAAEO94wR_rDPALyK1KYvtjA2CuuWM5A9_T9SfnRAF0OoZs-AABAAha6OA-Uw4nEHi3AleA-W59sVAjTvpPg1XtuFYEnVHG0TBqTabIrSuNIJP9OpSvkZiOWYRlPG839_wAPzU106Aw
          -JAB  -a-personal -FABELfzj-TkiKYWsNKk2WE8F8VEgbu3P-_HComVHcKrvGmY0AAAAAAAAAAAAAAAAAAAAAAAEO94wR_rDPALyK1KYvtjA2CuuWM5A9_T9SfnRAF0OoZs-AABAAFzLPWIG3XEXzKkGxIMAozgh9me6Ss2J4HZpdtqtZDLJRzic2wTePt4wx_PNgLcukTFe70-MXy2VlJvIaJtOsDA
```

```
{
  "v": "KERI10JSON00006a_",
  "t": "exn",
  "d": "EF3Dd96ATbbMIZgUBBwuFAWx3_8s5XSt_0jeyCRXq_bM",
  "dt": "2021-11-12T19:11:19.342132+00:00",
  "r": "/credential/issue",
  "a": {
    "v": "ACDC10JSON00011c_",
    "d": "EpcEvrX2gGTpmKbIG25GSA7_LsWwwzVQ6aUilgBubpGI",
    "i": "EmSIYYxvgtKn9jAp8GcK3fXOwTeyBIcAnRnyrLNfKjVI",
    "s": "EZdaE1HCu2ZhyIhpXTWfGSLS2kirKexaC-4up3sIUz1I",
    "a": {
      "d": "EG7PfBZbyXS0huxbnvMD-jOdTnDCCaT4CO6xId-ATNWg",
      "i": "EhwC_-ISX9helBDUSKU5j_VEU3G0Jp6ZFRD9dTb4-5c0",
      "dt": "2021-06-09T17:35:54.169967+00:00",
      "ri": "EymRy7xMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggt",
      "LEI": "254900YH3ZCDPE1E5306",
      "personal": {
        "d": "ExSxzzsvGlhO6W0O1_-k5Api8NgD8dDHLgazBxTqi_z4",
        "n": "Q8rNaKITBLLA96Euh5M5v4o3fRl1Bc54xdM-bOIHUjY",
        "personLegalName": "Anne Jones",
        "engagementContextRole": "Project Manager"
      }
    }
  }
}-AAB (message signature)
-KAC  -a  -JAB  -  -FABELfzj-TkiKYWsNKk2WE8F8VEgbu3P-_HComVHcKrvGmY0AAAAAAAAAAAAAAAAAAAAAAAEO94wR_rDPALyK1KYvtjA2CuuWM5A9_T9SfnRAF0OoZs-AABAAha6OA-Uw4nEHi3AleA-W59sVAjTvpPg1XtuFYEnVHG0TBqTabIrSuNIJP9OpSvkZiOWYRlPG839_wAPzU106Aw
          -JAB  -a-personal -FABELfzj-TkiKYWsNKk2WE8F8VEgbu3P-_HComVHcKrvGmY0AAAAAAAAAAAAAAAAAAAAAAAEO94wR_rDPALyK1KYvtjA2CuuWM5A9_T9SfnRAF0OoZs-AABAAFzLPWIG3XEXzKkGxIMAozgh9me6Ss2J4HZpdtqtZDLJRzic2wTePt4wx_PNgLcukTFe70-MXy2VlJvIaJtOsDA


```

# Presentation Exchange

# Selective Disclosure Credentials

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Security Considerations

TODO Security

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

Dr Sam Smith, Kevin Griffin and the Global Legal Entity Identifier Foundation (GLEIF)
