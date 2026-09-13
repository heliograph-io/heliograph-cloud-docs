# What is open source, what is fair source, and what is not either

heliograph is three licences across four components, and this page exists so you do not have to work out which is which from repository badges.

If you are deciding whether to depend on this, the short version is at the top and everything under it is the detail behind it.

## Read the tense before you read the table

**The licence set below is decided and is not yet applied.** `heliograph/LICENSE:1` and `heliograph-relay/LICENSE:1` both say "MIT License" this morning, and both repositories still carry an MIT badge (`heliograph/README.md:9`, `heliograph-relay/README.md:7`). So every row below is **what the announcement applies**, not what a `git clone` gives you today.

That distinction is the whole reason this page is written the way it is. Publishing a licence in the present tense before the file says it is the same class of mistake as writing **cannot** where the truth is **does not currently**, and it has already happened once on this project. The note goes when the `LICENSE` files change, and not a day before.

Two things on this page are true today rather than pending, and they are marked where they appear: the old MIT grant, which cannot be withdrawn, and this documentation's own CC BY 4.0 licence, which is in the repository you are reading.

## The short version

- **The CLI, the station, the wire format, the crypto and the interfaces become open source under Apache 2.0, permanently.** No employee cap, no revenue cap, no production-use cap, ever
- **The relay becomes fair source under `FSL-1.1-ALv2`.** Publicly readable, free to run for your own purposes, and each release converts to Apache 2.0 two years after it ships. What it bans is offering relay hosting as a competing commercial service
- **The broker is proprietary**, and is not published. It is many to many: routing, pairing, policy and orchestration across estates
- **heliograph cloud is proprietary**: tenancy, billing, console, policy, archive
- **This documentation is CC BY 4.0** and its source is public, including the pages about the proprietary parts. That one is in force now: the licence is in this repository

**Every transport shape works without paying**, on a relay you run yourself. Nothing that crosses the gap sits behind a paywall, and that is a commitment rather than a current state of affairs.

## The three tiers, named properly

The licence column is the announced set, not the file on disk. See the tense note above.

| tier | components | licence | what it means |
|---|---|---|---|
| **open source** | CLI, station payloads, wire format, Noise handshake, `Transport` and `Channel` interfaces | **Apache 2.0** | genuinely open source, OSI-approved, permanently. Fork it, sell it, embed it |
| **fair source** | the relay: one control to one station, carrying every shape | **`FSL-1.1-ALv2`** | publicly readable, free to use and self-host, **may not be offered as a competing commercial service**. Each release converts to Apache 2.0 two years after it ships |
| **proprietary** | the broker, and heliograph cloud | none published | many-to-many orchestration; tenancy, billing, console, policy, archive |

### "Fair source" is a real term and we are using it because it is precise

Fair Source is a defined initiative, not a phrase invented to avoid saying something worse. It means three things: the source is publicly readable, the restrictions are the minimum needed to protect the producer's business model, and there is **delayed open-source publication** on a stated clock. That last idea is itself an Open Source Initiative concept.

Saying "fair source" about the relay will be accurate once the licence lands. Saying "open source" about it would not be, and we are not going to.

**And calling the broker fair source would be just as wrong.** Its source is not published, and nothing about it converts to Apache 2.0 on any clock. It is proprietary. There is no third category being smuggled in here.

## The relay and the broker are two different things

This is the distinction the rest of the page rests on, and it is easy to blur.

```
   relay      one control  ──────────►  one station
              carries beacon, flare and beam
              fair source once the announcement lands
              always runnable by you

   broker     many controls ─────────►  many stations
              routing, pairing, policy, orchestration
              proprietary, hosted only
```

A **relay** is a queue with a token check that lets one control and one station reach each other when neither can reach the other directly. It carries every shape. You can always run one.

A **broker** is the augmentation on top: many controls, many stations, routing and policy in the middle. It is a multi-estate concern, which is the only class of thing this project puts behind a price.

## What you can run yourself, exactly

| capability | can you run it yourself? |
|---|---|
| **beacon** - a message left where both sides can see it, collected later | **yes**, and it is what ships today: git, relay, file share, object store, bundle, Azure Blob |
| the same, with **nothing compiled** on the far side | **yes** for git, file share, bundle and object store. The relay's bash station needs `heliograph-seal`; its PowerShell station needs nothing |
| **flare** - a single exchange fired at a station you can reach | **yes**, `intercom.sh` in the transport repo |
| **beam** - a live line held open in both directions | **when it exists.** It is designed and is not yet a transport you can pick (`site/content/transports.md:27-28`). When it ships it ships in the relay, unpaywalled |
| every passenger the beam will carry - shell, PTY, ssh passthrough | **with the beam**, same answer |
| the crypto, the gates, the key handling | **yes**, and always will be. Apache 2.0 |
| audit and log **generation** | **yes**, and never a paid feature |
| **the relay itself** | **yes**, for your own purposes, free, under FSL |
| **the broker** | **no.** Not now and not planned. It is the hosted product |
| multi-user identity, RBAC, approvals, policy plane | **no** |
| archive retention, cross-estate search, SIEM export | **no** |

**The beam row is written that way on purpose.** It would read better as "yes, complete", and that would be false: the beam is a design, not a shipped transport, and `site/content/transports.md:27-28` says so in the public docs already. A capability table that promised it would be the exact failure this page exists to prevent.

**And the second row is there because "beacon and flare need no binary" is not quite true**, which is a thing we have said and are correcting. The relay *is* a beacon, and its bash station needs `heliograph-seal`, because end-to-end encryption needs X25519, ChaCha20-Poly1305, Ed25519 and HKDF and `curl` and coreutils do not do those. So an estate that permits no compiled code anywhere loses **the relay on a bash station and the beam: two shapes, not the tool.** Everything else is complete.

The allowlist is a file rather than a policy note - `station/FAR-SIDE-BINARIES` in the public repository, read by CI, so a second binary is a deliberate edit to a visible list rather than a side effect. Adding one requires that the feature genuinely cannot be done in bash, that the build is reproducible so you can check the bytes, and that the station verifies the binary by hash before running it.

## Two sentences we are retiring, because they are false

**"The broker is open source."** It is not and will not be. The **relay** is fair source, converting to Apache 2.0 two years after each release. The **broker** is proprietary. If you read "the broker is open source" anywhere with our name on it, it is out of date and we would like to know where.

**"Self-host everything for nothing."** Nearly true and not true enough. Under FSL, running the relay for your own purposes is permitted and free, so the substance survives: you can reach your machines, capture your logs and keep your evidence without paying us anything. What is **not** self-hostable is the governance plane - multi-user identity, approvals, policy, cross-estate archive - and the broker itself. The capability table above is the honest form of that sentence and it replaces it.

## What we are committing to, in writing

**The Apache 2.0 components stay open source permanently**, named explicitly: the CLI, the station payloads, the wire format, the Noise handshake, and the `Transport` and `Channel` interfaces.

**No employee cap, no revenue cap, no production-use cap on them. Ever.** This is worth being specific about because the industry has a worked example of both behaviours. Portainer puts revenue caps on cheap *paid* tiers and nobody objects. Teleport put an employee-and-revenue cap on its *free open-source* edition and reviewers called it a betrayal of early adopters. **A cap on a paid tier is ordinary segmentation. A cap on the open-source edition is a trust event.** The FSL non-compete on the relay is the only restriction anywhere in this project, and it restricts competitors rather than users.

**If DBHQ ever relicenses or abandons the open components, the last open commit stands and DBHQ will not pursue forks.**

**The old MIT commits remain MIT and may be forked for ever.** Permissions already granted cannot be withdrawn, and we would rather say that here than let somebody discover it and wonder what else was not mentioned.

**The professional-services permission.** FSL has no Additional Use Grant, and "Competing Use" is ambiguous for a consultancy that self-hosts relays to work on client estates. A copyright holder may always grant more than the licence does, so:

> DBHQ additionally permits deploying the relay to reach estates you or your clients operate, as part of professional services, provided you do not offer relay hosting itself as a hosted or subscription service to third parties.

To be plain about where the line is: running relays to reach your own clients' machines is fine and is what the relay is for. Selling relay hosting to third parties is competing use, and it is the one use this grant deliberately does not reach.

**That sentence used to end "because hosted relay is a tier we sell", and it was withdrawn on 2026-09-13 rather than edited away.** Nothing is being sold. [`README.md`](../README.md) says no page here states a price, a tier, an allowance or a launch date; [the threat model](threat-model.md) says "it is why nothing is being sold yet"; and the public relay page says "Free today. No card, no account, and nothing to buy" (`site/content/relay.md:44`). A licence page is where somebody checks a claim before a procurement review, so a tier asserted here and contradicted in three other places is exactly the sentence this page exists to catch. **The grant does not depend on it** - a copyright holder may always permit more than the licence does, whether or not anything is for sale.

## The security argument does not depend on any of this

Worth stating on this page specifically, because somebody reading about a licence change will reasonably wonder what it costs them.

**The security argument needs *readable* source, not *free* source**, and FSL keeps the source readable. The relay's own package comment is the argument, and it is as checkable under FSL as it was under MIT:

> THERE IS NO KEY HERE WORTH STEALING AND NO PLAINTEXT TO SUBPOENA, and that is the design rather than an omission. Every message arrives already sealed by the client, bound to its estate, its station, its direction and its sequence number, and signed. This server sees a byte slice, a routing key and a length.

`heliograph-relay/relay.go:1-20`. You can read it in about five minutes and you do not need our permission to.

**And reading it now reaches further than it did.** Both relay implementations are **reproducibly built**, proved on every pull request rather than asserted: each is built twice from two directories, the second from a copy with no `.git`, and any difference fails the build. You can run `edge/reproduce.sh` or `packaging/reproduce.sh` against a tag and arrive at the hash we publish, and `GET /health` reports the version and hash of the artefact serving.

That matters here because it is the link between "you may read the source" and "the source you read is what is running", and a licence that keeps the source readable is worth more once that link exists. **The link is not complete**: nothing is signed, nothing is deployed, and a Worker cannot read its own code, so its reported hash is a number the deploy workflow computed. The [security page](security.md) says exactly where it stops. The licence does not change any of it in either direction.

**That comment used to open "THERE IS NO CRYPTOGRAPHY IN THIS PACKAGE", and it does not any more.** The change is on this page rather than left to somebody's diff, because that sentence was offered as a reason to trust the relay and a licence page is exactly where a reader checks whether what they were told still holds.

The relay now verifies authorisation lease signatures with Ed25519. **The proxy narrowed and the claim did not:** a public key is not a secret, so an attacker who takes everything the relay holds gets a key that checks signatures and makes none. HMAC-SHA256 would have passed the old rule as written and was rejected for being symmetric, because a relay able to verify would be able to **mint**. And the narrower rule is enforced by the linker rather than by a comment: `TestTheRelayBinaryCannotSignALease` reads the built binary's symbol table, and Go links only reachable code.

What it cost is stated in the relay's own README and is repeated here: **you can no longer confirm the old sentence with one `grep`.** The rule is stronger and less casually checkable. The [security page](security.md) carries the full reasoning.

## The part that is proprietary, and what that costs you

The broker is not published, and we are not going to pretend that is free of consequence.

**What protects you is architectural rather than legal: the broker never touches ciphertext.** It decides which queue a message lands in, who may collect it, and what policy applies. The bytes themselves move through relay code, which is published. So the thing that touches your content is readable, and the thing that is not readable never touches it.

That is a hard rule: the moment the broker buffers, transforms, inspects or re-frames a message, it breaks.

**And as of 2026-09-13 it is enforced by the type system rather than by anybody remembering.** The relay's authorisation surface (`heliograph-relay/auth.go:233-291`) is checked by a test that walks every parameter and every return value on it and fails on anything that could carry or yield bytes. An authoriser gets strings, numbers, times and structs of those. There is nothing it could follow to your content, and adding a field that would change that fails the build.

Its limit, stated because we would rather you knew: that constrains what the relay hands an authoriser. A component placed inline as a proxy would not be an authoriser and the test would not see it. Not doing that remains a rule rather than a mechanism.

**And here is the sentence we are not going to write.** "The unpublished component never handles your data" is false, and an adversarial review caught us drafting it. Account identifiers, station identities, IP addresses, topology, pairing events, timing and policy decisions are all your data, and the broker holds them. A broker in the authorisation path can also deny service, isolate a station, correlate activity, redirect a client, downgrade policy, and conceal that it did, because nobody can read it.

The accurate version:

> Payload content is end-to-end encrypted and passes only through published components. The proprietary broker handles account, topology and authorisation metadata, and can permit, deny or route a connection attempt. It cannot decrypt or forge a payload, and it takes no part in establishing identity - the fingerprint comparison that pairs two ends happens out of band between two people (`site/content/relay.md:96-97`).

**That last clause is the load-bearing one**, because a broker that could substitute a key while two ends were pairing would be fatal rather than merely opaque. It holds only while the operator actually compares the fingerprints. The public relay page calls that "the only step a machine cannot do for you" (`site/content/relay.md:97`), which is a **declaration and not an enforcement**: nothing stops an operator skipping it and nothing detects that they did.

What the broker retains, per field, with retention periods, is on the [security page](security.md). We publish that list rather than leave you to derive it.

## If a third party holding this is outside your threat model

Then run the relay yourself. That is the honest answer and it is not a deflection: the relay is the component in the data path, you can always run it, and nothing about running it is degraded.

What self-hosting the relay does **not** remove is the archive, which is the proprietary part and the reason the service is worth paying for. The [security page](security.md) says so in those words rather than letting "self-hostable" do more work than it can carry.

## Where to challenge this

The source of this page is public and takes pull requests. If a claim here does not match the code, that is a defect and we would rather have it as a pull request than as a surprise in a procurement review.
