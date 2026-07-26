# Tikpema — ERC-8004 identity documents (public mirror)

This repository publishes **two** frozen ERC-8004 identity documents, so that anyone can reproduce
each one's content address from source and check it against what is anchored on-chain.

| document | subject | agentId |
| --- | --- | --- |
| [`unified.json`](./unified.json) | **Tikpema Agent** — the custodial agent system that moves funds | `851823` |
| [`dd-service.json`](./dd-service.json) | **Tikpema DD Service** — read-only on-chain due diligence | `851891` |

**Both identities are owned by the same wallet**, so the owner discriminates nothing between them
and the `tokenURI` is what tells them apart — see **Two identities, one owner** below.

Everything from here down to *Tikpema DD Service* concerns [`unified.json`](./unified.json) and
`agentId 851823`. The DD-service document has its own section after it.

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

## ✅ Status: REGISTERED ON-CHAIN — `agentId 851823`

The document has been registered. The canonical identity is:

```
agentId    851823
Registry   0x8004A818BFB912233c491871b3d84c89A494BD9e   (IdentityRegistry)
Network    Arc Testnet, chainId 5042002
```

**Both self-referential invariants were confirmed by a read-only `eth_call` against the registry:**

```
ownerOf(851823)  == 0xc54d47211997aca90ef4fcfbc742a3b511b4e621
tokenURI(851823) == ipfs://bafkreidoeond3akvswce3e425o5grfygsvrfyleqkwathio4ae6y6vujae
```

The `tokenURI` equals the CID at the top of this file, which closes the document's central claim
(`tokenURI(agentId) == <the CID of this exact document>`). **The full verification chain can now be
walked end to end** — see [How to verify](#how-to-verify) below; you no longer have to stop at the
content-address check.

Verify it yourself, trusting no server we run:

```sh
# read the on-chain pointer directly — substitute any Arc Testnet RPC
cast call 0x8004A818BFB912233c491871b3d84c89A494BD9e \
  "tokenURI(uint256)(string)" 851823 --rpc-url https://rpc.testnet.arc.network
# expect ipfs://bafkreidoeond3akvswce3e425o5grfygsvrfyleqkwathio4ae6y6vujae

cast call 0x8004A818BFB912233c491871b3d84c89A494BD9e \
  "ownerOf(uint256)(address)" 851823 --rpc-url https://rpc.testnet.arc.network
# expect 0xc54d47211997aca90ef4fcfbc742a3b511b4e621
```

**This repository is still a mirror and still carries no authority** (see below): the on-chain
`tokenURI` is what proves the identity, not this file. What changed is that there is now an
`agentId` to point at, and it resolves to these exact bytes.

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
`0x7274e874ca62410a93bd8bf61c69d8045e399c02`, read from the EIP-1967 implementation slot and
confirmed unchanged on 2026-07-24). Its upgrade authority can change the registry's behaviour,
including in principle what `tokenURI` and `ownerOf` return. We neither control that registry nor
represent its upgrade behaviour as fixed. This is disclosed for the same reason the agent's own
due-diligence tooling flags owner-power on third-party contracts.

> ### ⚠️ The empty admin slot does NOT mean "not upgradeable"
>
> If you check the standard EIP-1967 **admin** slot
> (`0xb53127684a568b3173ae13b9f8a6016e243e63b6e8ee1178d6a717850b5d6103`) on this registry, it reads
> **all zeros**. That is easy to misread as "no admin, therefore immutable." **It is not.**
>
> This is not a Transparent proxy, where an admin address is stored at that slot. The
> implementation slot is populated (see above) and the proxy itself is only ~130 bytes of
> delegating bytecode, which is the UUPS pattern: **upgrade authority lives inside the
> implementation contract**, typically behind an owner or role check, not at the admin slot. An
> empty admin slot is what UUPS looks like — it is the absence of a *storage location*, not the
> absence of *upgrade power*.
>
> So: **do not conclude immutability from a zero at that slot.** To assess who can upgrade this
> registry, inspect the implementation's own authorisation logic. We are pointing this out
> precisely because the naive check returns a reassuring-looking answer that is wrong, and we would
> rather you not be misled by it in our favour.

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

**Complete the chain (`agentId 851823`):**

1. Read `tokenURI(851823)` from the IdentityRegistry on-chain — expect
   `ipfs://bafkreidoeond3…` (confirmed on-chain; see the command in the Status section).
2. Fetch the bytes at that `ipfs://` CID.
3. Hash the bytes and confirm they match the CID.
4. Trust *that* text — not any server response, including this repository.

If step 1 returns a CID other than `bafkreidoeond3…`, then this mirror is stale or superseded and
**the on-chain value wins.**

### Identifying the canonical identity

The document cannot name its own `agentId` — the id is minted by registering the document as
`metadataUri`, so the id did not exist until after the bytes were fixed. The canonical identity is
**`agentId 851823`**, being the one that satisfies **both** conditions:

```
ownerOf(851823)  == 0xc54d47211997aca90ef4fcfbc742a3b511b4e621
tokenURI(851823) == bafkreidoeond3akvswce3e425o5grfygsvrfyleqkwathio4ae6y6vujae
```

**The `tokenURI` half is the discriminator.** Two earlier orphan identities — **`602428` and
`850337`** — are owned by that same wallet but carry a *different* `tokenURI`
(`bafkreibdi6623n…`, the quickstart document). Anyone scanning the registry by owner address will
find all three; owner alone does not identify the canonical record, and only `851823` carries the
`tokenURI` above.

The address `0xc85b3d9BdEb3703c5778E817b8bC30c96f1cB006` appears in older material as a
"cold owner". **It was never the on-chain owner. It is stale and must not be used.**

---

# Tikpema DD Service — [`dd-service.json`](./dd-service.json)

```
sha256  d3734accb6390a361df2daf87b49c41d4a44d30bfc9285f47be3c3284dbb402f
CID     bafkreigtonfmznrzbi3b34w27b5utra5jjcngc74skc7i67dymue3o2af4
        (CIDv1, raw codec, 28628 bytes)
IPFS    ipfs://bafkreigtonfmznrzbi3b34w27b5utra5jjcngc74skc7i67dymue3o2af4
```

## What this document is

The ERC-8004 identity metadata for the **Tikpema DD Service**, version **1.0.0**. In the document's
own words, it is *"a read-only on-chain due-diligence service. Given a contract address on Arc, it
returns an INVENTORY of what the contract's privileged holder can do to a depositor — enumerated
from the deployed bytecode — together with a machine-generated manifest of what it did NOT check.
It emits no verdict, no score and no recommendation."*

Its reputation subject is therefore **report accuracy and honest bounding — not "verdict accuracy",
because the service emits no verdicts.** Scoring it on verdicts would invent a product it does not
offer.

The document publishes **three** distinct independence limits rather than one, and they are worth
reading before trusting any report it produces:

1. **Operator** — same operator as the Tikpema agent. *"This is not an independent auditor."*
2. **Code** — *"the auditor shares code with the audited system."* The analyzer and the production
   deposit gate both import the same fact-production primitive, so a blind spot in the shared
   catalogue is invisible to **both at once**.
3. **Data source** — multi-endpoint quorum independence is *asserted, not proven*; every report
   carries `independenceVerified: false`.

## ✅ Status: REGISTERED ON-CHAIN — `agentId 851891`

The document has been registered. The canonical identity is:

```
agentId    851891
Registry   0x8004A818BFB912233c491871b3d84c89A494BD9e   (IdentityRegistry)
Network    Arc Testnet, chainId 5042002
tx         0xd33cb296ba2dcc68c29e29cef055f9b959973b11eea3d0a97dadfa9437db20f1
```

**Confirmed by a read-only `eth_call` against the registry:**

```
ownerOf(851891)  == 0xc54d47211997aca90ef4fcfbc742a3b511b4e621
tokenURI(851891) == ipfs://bafkreigtonfmznrzbi3b34w27b5utra5jjcngc74skc7i67dymue3o2af4
```

The `tokenURI` equals the CID at the top of this section, which closes the document's central claim
(`tokenURI(agentId) == <the CID of this exact document>`).

Verify it yourself, trusting no server we run:

```sh
# read the on-chain pointer directly — substitute any Arc Testnet RPC
cast call 0x8004A818BFB912233c491871b3d84c89A494BD9e \
  "tokenURI(uint256)(string)" 851891 --rpc-url https://rpc.testnet.arc.network
# expect ipfs://bafkreigtonfmznrzbi3b34w27b5utra5jjcngc74skc7i67dymue3o2af4

cast call 0x8004A818BFB912233c491871b3d84c89A494BD9e \
  "ownerOf(uint256)(address)" 851891 --rpc-url https://rpc.testnet.arc.network
# expect 0xc54d47211997aca90ef4fcfbc742a3b511b4e621
```

> ### ⚠️ The document says "NOTHING REGISTERED" — that is expected, not a contradiction
>
> `dd-service.json` states: `NO WALLET CREATED, NOTHING REGISTERED, NO agentId EXISTS FOR THIS
> SERVICE`. **That was true when the bytes were frozen, and it can never be updated.** The document
> is content-addressed, so "correcting" it would change its CID and thereby break the very claim its
> `tokenURI` anchors.
>
> The registration is recorded **here instead** — which the document itself designates: *"the owner
> is recorded in the public mirror's README after registration, which is outside the CID precisely
> so it can be."* Where the frozen text and this section differ on registration status, the frozen
> text is a **snapshot taken before registration** and this section is current. Nothing about the
> engine's described behaviour, limits or coverage is affected.

## Reproduce the content address from source

```sh
curl -sL https://raw.githubusercontent.com/tikpema274/tikpema-agent-identity/main/dd-service.json -o dd-service.json

sha256sum dd-service.json
# expect d3734accb6390a361df2daf87b49c41d4a44d30bfc9285f47be3c3284dbb402f

ipfs add --only-hash --cid-version=1 --raw-leaves dd-service.json
# expect bafkreigtonfmznrzbi3b34w27b5utra5jjcngc74skc7i67dymue3o2af4
```

**The same flag caveat applies as for `unified.json`** — `--cid-version=1 --raw-leaves` are
required, and a `bafybei…` or `Qm…` result means the invocation differed, not that the file is
wrong. See the warning in [How to verify](#how-to-verify) above.

**Fetch independently from IPFS** — via any gateway, or your own node:

```sh
curl -sL https://ipfs.io/ipfs/bafkreigtonfmznrzbi3b34w27b5utra5jjcngc74skc7i67dymue3o2af4 | sha256sum
```

## Identifying the canonical identity — `tokenURI` is the *only* discriminator

As with the agent document, `dd-service.json` cannot name its own `agentId`: the id is minted by
registering the document, so it did not exist until after the bytes were fixed.

**But the test here is one-sided, and deliberately so.** `dd-service.json` **declares no owner
address at all** — when it was frozen, the wallet decision had not been made, and naming one would
have asserted something not yet true. So:

```
tokenURI(851891) == bafkreigtonfmznrzbi3b34w27b5utra5jjcngc74skc7i67dymue3o2af4
```

**is the whole test.** That single condition is sufficient, because the CID is a pure function of
these bytes — no other document can satisfy it. Do **not** expect a two-condition
`owner + tokenURI` check from these bytes the way `unified.json` supports one; the owner is
recorded in this README precisely because it is outside the CID.

# Two identities, one owner — and why that is stated plainly

`851823` and `851891` are owned by **the same wallet**:

```
ownerOf(851823) == ownerOf(851891) == 0xc54d47211997aca90ef4fcfbc742a3b511b4e621
```

**The owner therefore identifies nothing.** Anyone scanning the registry by owner address will find
`851823`, `851891`, and two earlier orphans (`602428`, `850337`) — four identities, one address.
Only the `tokenURI` separates them:

| agentId | tokenURI | document |
| --- | --- | --- |
| `851823` | `bafkreidoeond3…` | `unified.json` — the agent |
| `851891` | `bafkreigton…` | `dd-service.json` — the DD service |
| `602428`, `850337` | `bafkreibdi6623n…` | orphans (quickstart document) |

This is **shared operator, separate subjects**, and the DD document states the consequence directly:

> *"That identity's reputation concerns an agent that MOVES FUNDS — swaps, bridges, vault deposits,
> x402 purchases. This service's reputation concerns whether its READ-ONLY REPORTS were accurate and
> honestly bounded. Good swap execution is not evidence of good reporting and vice versa. The two
> must not be merged into one score, and a reader should not transfer trust between them."*

So: **do not transfer trust between these two identities.** A record of correct fund movement says
nothing about report accuracy, and vice versa. They share an operator and a wallet — nothing more.

---

## The pointer is mutable — and that is disclosed, not hidden

The identity owner can call `setAgentURI(agentId, newCid)` and re-point the identity at any time.
This document does **not** claim otherwise. What remains true regardless:

- Each CID is content-addressed and immutable — the bytes hash to it or they are not the document.
- Every re-point is an **on-chain event**: supersessions are publicly visible, never silent.
- Verification is server-independent, so it does not depend on our good behaviour.

## Editing rules

**Neither `unified.json` nor `dd-service.json` MAY BE EDITED. Not one byte. Either of them.**

Each document is **self-referential**: it asserts that `tokenURI(agentId)` equals the CID of *that
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

**`README.md` is NOT part of any CID** and can be edited freely — correcting, expanding, or
updating this text has no effect on either document's content address. That asymmetry is
deliberate: everything that must stay fixed lives in the two frozen documents, and everything
explanatory lives here. It is also load-bearing — `dd-service.json` declares no owner and was
frozen before registration, so this README is where its owner and `agentId` are recorded, by the
frozen document's own explicit design.

## Provenance

This is a **mirror**. The working source is a private repository, which remains the place the
document is authored and versioned; this repository exists only to make the artifact
independently retrievable and the CID independently reproducible. If the two ever disagree, the
on-chain `tokenURI` is authoritative over both.
