# Tikpema Agent — ERC-8004 identity document (public mirror)

This repository publishes one file, [`unified.json`](./unified.json), so that anyone can
reproduce its content address from source and check it against what is anchored on-chain.

```
sha256  6e239a3d815595844d939aebba68970695625c2c90558133a1dc013d8f568901
CID     bafkreidoeond3akvswce3e425o5grfygsvrfyleqkwathio4ae6y6vujae
        (CIDv1, raw codec, 13656 bytes)
IPFS    ipfs://bafkreidoeond3akvswce3e425o5grfygsvrfyleqkwathio4ae6y6vujae
```

## What this document is

The ERC-8004 identity metadata for **Tikpema Agent**, version **2.0.0**: one custodial agent
system on Arc, operated as a single identity. Four internal roles (Researcher, Second Opinion,
Executor, Vault) share one Circle developer-controlled agent wallet, one set of fail-closed
spending caps, and one operator. The roles are attribution and kill-switch boundaries — they are
**not** independent actors and not separate reputation subjects.

The document states its own custody model plainly: the identity is **self-owned by the agent's
custodial Circle developer-controlled wallet**. The operator, through Circle, can move or re-point
it. There is no offline cold key. Ownership alone is not tamper-evidence — the tamper-evidence
comes from content-addressing plus on-chain anchoring, which is what this repository lets you check.

## ⚠️ Status: NOT YET REGISTERED ON-CHAIN

**No `agentId` exists yet.** The document asserts `tokenURI(agentId) == <the CID above>`, but
registration has not happened, so **there is currently no on-chain id to query and the verification
chain below cannot be completed end to end.**

What that means for a reader, stated bluntly:

- You **can** verify today that these bytes hash to `6e239a3d…` and address to `bafkreidoeond3…`.
  That is a real, self-contained check and it is the reason this mirror exists.
- You **cannot** yet verify that any on-chain identity points at this document, because none does.
- **This repository is therefore a source copy, not proof of an on-chain identity.** Do not read
  its existence as evidence that the agent is registered. It is not.

This section will be updated with the concrete `agentId` once registration occurs. Until you see
one here *and* can confirm it yourself against the registry, assume unregistered.

## The authoritative pointer is on-chain, not here

**This repository is a MIRROR and carries no authority.** A copy of a file on GitHub proves
nothing about what an agent's identity resolves to — it can be edited, deleted, or served by
anyone. Once registration has happened, the only claim worth anything is:

> `tokenURI(agentId)` read directly from the ERC-8004 IdentityRegistry.

```
Registry   0x8004A818BFB912233c491871b3d84c89A494BD9e   (IdentityRegistry)
Network    Arc Testnet, chainId 5042002
```

**Note the network: Arc *Testnet*, not mainnet.** Do not assume a mainnet deployment.

**The registry is an upgradeable EIP-1967 proxy** (implementation
`0x7274e874ca62410a93bd8bf61c69d8045e399c02` as of 2026-07-23). Its admin can change the
registry's behaviour, including in principle what `tokenURI` and `ownerOf` return. We neither
control that registry nor represent its upgrade behaviour as fixed. This is disclosed for the same
reason the agent's own due-diligence tooling flags owner-power on third-party contracts.

## How to verify

Content addressing means the bytes hash to the CID or they are not this document. Nothing in this
check requires trusting us, this repository, or any server we run — which is precisely why it is
the only check worth performing.

**Reproduce the content address from source (works today):**

```sh
curl -sL https://raw.githubusercontent.com/tikpema274/tikpema-agent-identity/main/unified.json -o unified.json

sha256sum unified.json
# expect 6e239a3d815595844d939aebba68970695625c2c90558133a1dc013d8f568901

ipfs add --only-hash --cid-version=1 --raw-leaves unified.json
# expect bafkreidoeond3akvswce3e425o5grfygsvrfyleqkwathio4ae6y6vujae
```

> ### ⚠️ Use those exact flags, or you will see a false mismatch
>
> **`--cid-version=1 --raw-leaves` are required.** They select the raw-codec (`bafkrei…`) form,
> which is the CID this document asserts about itself and the form ERC-8004 metadata URIs use.
>
> **Plain `ipfs add unified.json` returns a completely different-looking CID** — a `Qm…` (CIDv0)
> or, with `--cid-version=1` alone, a **`bafybei…` dag-pb CID**. That is IPFS wrapping the bytes in
> a UnixFS/dag-pb node and addressing the *wrapper*, not the raw bytes. **It does not mean the file
> is wrong.** The bytes can be perfectly correct and still produce `bafybei…` under the wrong flags.
>
> So: a `bafybei…` or `Qm…` result is **not evidence of tampering** — it means the invocation
> differed. Re-run with both flags before concluding anything. Conversely, if you get
> `bafkreidoeond3…` with the correct flags, the bytes are exactly right.
>
> `--only-hash` merely computes the address without storing the file in your local node; drop it if
> you also want to pin your own copy. It has no effect on the resulting CID.

The CID is a pure function of the bytes: multibase `b` + base32 of
`0x01 | 0x55 | 0x12 | 0x20 | <32-byte sha256>` (CIDv1, raw codec, sha2-256). You can derive it
from the sha256 alone, without IPFS installed — so the sha256 check and the CID check are two
views of one fact, and **the sha256 comparison alone is sufficient** if you would rather not
install IPFS at all.

**Fetch independently from IPFS** — via any gateway, or your own node:

```sh
curl -sL https://ipfs.io/ipfs/bafkreidoeond3akvswce3e425o5grfygsvrfyleqkwathio4ae6y6vujae | sha256sum
```

**Complete the chain (once an `agentId` exists):**

1. Read `tokenURI(agentId)` from the IdentityRegistry on-chain.
2. Fetch the bytes at that `ipfs://` CID.
3. Hash the bytes and confirm they match the CID.
4. Trust *that* text — not any server response, including this repository.

If step 1 returns a CID other than `bafkreidoeond3…`, then this mirror is stale or superseded and
**the on-chain value wins.**

### Identifying the canonical identity

The document cannot name its own `agentId` — the id is minted by registering the document as
`metadataUri`, so the id does not exist until after the bytes are fixed. The canonical identity is
therefore whichever satisfies **both** conditions:

```
ownerOf(agentId)  == 0xc54d47211997aca90ef4fcfbc742a3b511b4e621
tokenURI(agentId) == bafkreidoeond3akvswce3e425o5grfygsvrfyleqkwathio4ae6y6vujae
```

**The `tokenURI` half is the discriminator.** Two earlier orphan identities are owned by that same
wallet but carry a *different* `tokenURI`. Anyone scanning the registry by owner address will find
them; owner alone does not identify the canonical record.

The address `0xc85b3d9BdEb3703c5778E817b8bC30c96f1cB006` appears in older material as a
"cold owner". **It was never the on-chain owner. It is stale and must not be used.**

## The pointer is mutable — and that is disclosed, not hidden

The identity owner can call `setAgentURI(agentId, newCid)` and re-point the identity at any time.
This document does **not** claim otherwise. What remains true regardless:

- Each CID is content-addressed and immutable — the bytes hash to it or they are not the document.
- Every re-point is an **on-chain event**: supersessions are publicly visible, never silent.
- Verification is server-independent, so it does not depend on our good behaviour.

## Editing rules

**`unified.json` MUST NOT BE EDITED. Not one byte.**

The document is **self-referential**: it asserts that `tokenURI(agentId)` equals the CID of *this
exact document*. The CID is a pure function of the bytes, so **any** change makes its own central
claim false — including changes that look like nothing at all:

- reformatting or re-indenting
- a `JSON.parse` → `JSON.stringify` round-trip
- converting LF to CRLF (this file is pure LF, 0 CR)
- adding or removing the trailing newline (the final byte `0x0a` is part of the frozen bytes)

Such a file would still parse, still look correct, and produce a different CID. **New versions
supersede with a new CID; they never edit in place.** This repository is committed with
`* -text` in `.gitattributes` specifically so that git cannot normalise line endings on any
platform.

**`README.md` is NOT part of the CID** and can be edited freely — correcting, expanding, or
updating this text has no effect on the document's content address. That asymmetry is deliberate:
everything that must stay fixed lives in `unified.json`, and everything explanatory lives here.

## Provenance

This is a **mirror**. The working source is a private repository, which remains the place the
document is authored and versioned; this repository exists only to make the artifact
independently retrievable and the CID independently reproducible. If the two ever disagree, the
on-chain `tokenURI` is authoritative over both.
