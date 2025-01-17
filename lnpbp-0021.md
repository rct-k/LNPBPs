```
LNPBP: 0021
Vertical: Smart contracts
Title: RGB non-fungible assets interface for collectibles (RGB-21)
Authors: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>,
         Hunter Trujillo,
         Federico Tenga,
         Zoe Faltibà,
         Carlos Roldan,
         Olga Ukolova,
         Giacomo Zucco,
Comments-URI: <https://github.com/LNP-BP/LNPBPs/issues/70>
Status: Proposal
Type: Standards Track
Created: 2020-09-10
License: CC0-1.0
```

- [Abstract](#abstract)
- [Background](#background)
- [Motivation](#motivation)
- [Design](#design)
- [Specification](#specification)
- [Compatibility](#compatibility)
- [Rationale](#rationale)
- [Reference implementation](#reference-implementation)
- [Acknowledgements](#acknowledgements)
- [References](#references)
- [Copyright](#copyright)
- [Test vectors](#test-vectors)


## Abstract


## Background


## Motivation


## Design

- [x] Media as URI (which can be attachments distributed with Storm or usual URLs)
- [x] Small media ("previews" if given together with large media)
- [x] Fraction of assets
- [x] Engravings
- [x] Reserves
- [x] Possible decentralized issue


## Specification

Interface specification is the following Contractum code:

```haskell
-- # Defining main data structures

-- collectibles are usually scarse, so we limit their max number to 64k
data ItemsCount :: U32

-- each collectible item is unique and must have an id
data TokenId :: U16

-- Collectibles are owned in fractions of a single item; here the owned
-- fraction means `1/F`. Ownership of the full item is specified `F=1`;
-- half of an item as `F=2` etc.
data OwnedFraction :: U64

data TxOut :: txid [Byte ^ 32], vout U16

-- allocation of a single token or its fraction to some transaction output
data Allocation :: TokenId, OwnedFraction

data Engraving :: appliedTo TokenId, content Media

data Media ::
    type MimeType,
    data [Byte]

data Attachment ::
    type MimeType,
    digest: [U8 ^ 32] -- this can be any type of 32-byte hash, like SHA256(d), BLACKE3 etc

data POR :: -- proof of reserves
    reserves TxOut,
    proof [Byte] -- schema-specific proof 

data Token ::
    id TokenId,
    name [Ascii ^ 1..40],
    details [Unicode ^ 40..256]?,
    preview Media?, -- always embedded preview media < 64kb
    media Attachment?, -- external media which is the main media for the token
    attachments { U8 -> ^ ..20 Attachment } -- auxiliary attachments by type (up to 20 attachments)
    reserves POR? -- output containing locked bitcoins; how reserves are
                  -- proved is a matter of a specific schema implementation

data Denomination :: 
    ticker [Ascii ^ 1..8],
    name [Ascii ^ 1..40],
    details [Unicode ^ 40..256]?,
    contract [Unicode]??,

 -- each attachment type is a mapping from attachment id 
 -- (used as `Token.attachments` keys) to a short Ascii string
 -- verbally explaining the type of the attachment for the UI
 -- (like "sample" etc).
data AttachmentType :: id U8, description [Ascii ^ 1..20]


interface RGB21
    global Name :: Denomination
    global Tokens :: Token+
    global Engravings :: Engraving+
    global isFractional :: Bool
    global AttachmentTypes :: AttachmentType+

    owned Allocations+ :: Allocation
    owned IssueRight* :: Amount
    owned DenominationRight?
    -- returns information about known circulating supply
    read supplyKnown :: Amount
        count Self.state["Tokens"]
        -- sum Self.ops["issue"]..closed["usingRight"].state ?? 0

    -- maximum possible asset inflation
    read supplyLimit :: Amount
        sum Self.IssueRight..state ?? 0

    read isCirculationKnown :: Bool
        all Self.ops["issue"]..stateKnown

    op transfer    :: inputs [Allocation+] -> beneficiaries [Allocation]
                   !! -- options for operation failure:
                      inequalFractions(TokenId)
                    | nonFractionalToken

    -- question mark denotes optional operation, which may not be supported by 
    -- some of schemata implementing the interface

    op? engrave    :: Allocation -> Allocation
                   <- Engraving

    op? issue      :: IssueRight -> IssueRight, beneficiaries {Allocation}
                   <- tokens {Token}, newAttachmentTypes {attachmentType}*
                   !! invalidReserves(POR)

    -- decentralized issue
    op? decentralizedIssue -> tokens {Token}, beneficiaries {Allocation}
                    !! invalidReserves(POR)

    op? rename :: DenominationRight -> DenominationRight? <- Denomination
                   <- Nomination
```

## Compatibility


## Rationale


## Reference implementation


## Acknowledgements


## References


## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.
