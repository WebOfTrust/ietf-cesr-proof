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
  SAI:
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


tags: IETF, CESR, SAID, KERI, ACDC

--- abstract

CESR Proof Signatures are an extension to the Composable Event Streaming Representation (CESR) that provide transposable
cryptographic signature attachments on self-addressing data (SAD).  A SAD (such as an ACDC Verifiable Credential) may be signed
with a CESR Proof Signature and streamed alongside any other CESR content.  In addition, a signed SAD can be embedded inside another
SAD and the CESR proof signature attachment can be transposed to the outer SAD and streamed without losing any cryptographic integrity.


--- middle

# Introduction

What problem are we solving.  How does it fit with CESR, KERI and SAIDs

## CESR Attachments


The specification adds 2 *Counter Four Character Codes* to the Master Code Table [TODO: add reference here] and 1 *Small Variable Raw Size Code Type*
to the Master Code Table (which results in 3 new code entries):

|   Code   | Description                                                                         | Code Length | Count or Index Length | Total Length |
|:--------:|:----------------------------------|:------------:|:-------------:|:------------:|
|          |                        **Counter Four Character Codes**                          |             |              |              |
|   -K##   | Count of attached qualified Base64 indexed controller signatures                    |      4      |       2      |       4      |
|   -J##   | Count of attached qualified Base64 indexed witness signatures                       |      4      |       2      |       4      |
|          |                        **Small Variable Raw Size Code**                          |             |              |              |
|   4A##   | Size of SAD Path with 0 Ante Bytes                                                  |      4      |       2      |       4      |
|   5A##   | Said of SAD Path with 1 Ante Byte                                                   |      4      |       2      |       4      |
|   6A##   | Size of SAD Path with 2 Ante Bytes                                                  |      4      |       2      |       4      |

## Additional Count Codes

## Streamable SADs

## Nested Partial Signatures

## Transposable Signature Attachments


# CESR Signatures

Introduction to CESR Signatures

## Signatures with Non-Transferable Identifiers

## Signatures with Transferable Identifiers

# CESR Sad Path Language

## Description and Usage

## CESR Encoding for SAD Path Language

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
Since the proof is a CESR attachment, this message cannot simply be embedded in another JSON document.  Therefore, a
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


# Hidden Attribute Presentations



# Security Considerations

TODO Security

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments

{:numbered="false"}

TODO acknowledge.
