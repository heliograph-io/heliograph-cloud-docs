# The threat model, in one table

Every claim Heliograph Cloud makes, the mechanism that enforces it, and the condition under which it fails.

This page exists because in this market the claim is the product, and the fastest way to lose it is to write **cannot** where the truth is **does not currently**. So there is a status column, and three of the rows below say the claim is one we have stopped making.

If you are evaluating this adversarially, this is the page to read.

## How to read it

| status | means |
|---|---|
| **holds** | a mechanism enforces it and a named test proves it |
| **conditional** | a mechanism enforces it, and something outside the mechanism has to stay true. The condition is named |
| **owed** | we intend it and nothing enforces it yet |
| **retired** | a sentence we used to write and have stopped, because it was not true |

Every citation is a path and a line in a public repository. Line numbers move, so each was read at `heliograph` `59d43ef` and `heliograph-relay` `3756eb7`, on 2026-09-13. If you find one that resolves to something unrelated, that is drift and a pull request against this page is welcome. If you find one that is simply absent, that is worse and we would like to hear about it.

**Drift is not hypothetical here and the dangerous case is not the broken link.** Re-reading every citation on this page against those two commits found six that still resolved to a line, and four of those resolved to a plausible neighbouring line about the same subject - which reads as correct and is not. One pointed at a paragraph on relay pricing where the page claimed a statement about fingerprint comparison. A citation that cannot be resolved announces itself; a citation that has slid eleven lines does not.

## The table

| claim | status | mechanism | fails when |
|---|---|---|---|
| the cloud cannot forge a request a station will accept, on the relay transport | **conditional** | `internal/seal/seal.go:262`, `internal/seal/seal.go:355`; `TestARelayCannotForgeARequest` | it administers the trusted set, or it obtains an identity file |
| the cloud never administers a station's trusted set | **conditional** | `internal/trust/apply.go:116`, `internal/trust/apply.go:22`; `internal/wire/status.go:41-43`; `internal/cloud/authorship_test.go:69` | nobody reads the set the station publishes |
| the cloud cannot read a message in transit on the relay | **holds** | `heliograph-relay/relay.go:1-7`, `heliograph-relay/relay.go:57`; `TestTheCiphertextRevealsNothing` | a customer's identity file reaches us |
| the cloud reads your logs in plaintext, in the archive | **not a cannot.** The trade | `station/bash/caplib.sh:205-220` bounds it, best effort | always. It is the product |
| a replayed message is refused, across a station restart | **conditional** | `internal/seal/seal.go:184`; `station/bash/transports/relay.sh:259-266` | the station's sequence state is lost or truncated |
| the signature covers what will run | **retired.** It covers which step is **named** | `internal/seal/seal.go:261`; `internal/wire/request.go:32-40` | always, for step content |
| `read-only` means the step changed nothing | **retired** | `station/bash/run.sh:260` reads a declaration | always. The gate catches the mistake, not the adversary |
| a gap in the timestamps proves silence at the point of capture | **holds, and only that** | `station/bash/caplib.sh:385-393` | never, for what it claims. It claims less than readers assume |
| the archive is complete | **retired** | none | always. Loss is detectable, not preventable |
| the broker never touches ciphertext | **holds, at the interface** | `heliograph-relay/auth.go:233-291`; `TestAnAuthoriserCannotObtainAMessageBody` | something other than an authoriser is put in the path |
| the relay we deploy is the relay you can read | **owed.** It reports what it believes it is | `heliograph-relay/edge/src/worker.ts:1128-1135` | a hand deploy. It is detection, not provenance |
| the claims above hold on every transport | **false as stated.** They hold on the relay | only `station/bash/transports/relay.sh` seals | git, object store, Azure Blob, file share and bundle carry unsigned plaintext |

Twelve rows. **Three hold outright, four hold conditionally, three are retired, one is owed, and one is false as commonly stated.** That is the honest position and it is why nothing is being sold yet.

**Two rows moved on 2026-09-13, and neither moved to "holds" without a condition.** The trust-root row went from **owed** to **conditional**, and the section below says exactly what the condition is. The broker row went from **owed** to **holds at the interface**, and the rest of this section is how.

Both are worth showing rather than announcing, because "we built it" is not evidence. The relay's authorisation surface - `Admission`, `Accounting`, `Sessions` and `Authoriser` at `heliograph-relay/auth.go:233-291` - is now checked by a test that walks every parameter and every return value on it by reflection and fails on anything that could carry, reference or yield bytes: a `[]byte`, a `Message`, an `io.Reader`, a `func`, a pointer, an `any`. An authoriser is handed strings, numbers, times and structs of those, and there is nothing it could follow to content.

That is a proof in the type system rather than a rule in a document, which matters because a rule saying "an authoriser must not look at bodies" is worth nothing: somebody adds a field for a good reason, nothing fails, and the claim quietly stops being true.

**Its limit, which is why the row says "at the interface".** The test constrains what the relay hands an authoriser. It does not constrain a future component placed inline as a proxy, because such a component would not be an authoriser and the test would not see it. The rule against that is still a rule.

## The four that need more than a row

### The trust root, which is the one that matters most

Signature verification only means a station trusts whatever key it was told to trust.

```
   the claim:     cloud has no signing key  ─────►  cloud cannot cause execution
   the gap:       cloud administers trust roots ──►  cloud installs its own key
                                               ──►  cloud signs legitimately
                                               ──►  station verifies happily
```

Nothing is stolen. No signature is forged. The first claim collapses anyway.

**This row said "owed, and it is a discipline you cannot check" until 2026-09-13.** It is now **conditional**, and the change is recorded here rather than applied quietly, because a page whose value is its status column has to show its own movement. The three things this section previously named as what would move it were: the station publishing its trusted set, a change to that set being itself a signed request, and a change being visible as an event rather than as configuration drift. Two of the three landed whole and the third landed partly.

**A change to the set is itself a signed document.** `internal/trust/apply.go:116` verifies it against the set as it stands, and `internal/trust/apply.go:83-85` refuses one that is not the next serial, so a change cannot be replayed or reordered. `internal/trust/set.go:90-92` keeps the member list unexported behind a single mutator, which is the compiler-enforced half rather than a reviewer's.

**The anchor is changeable only on the machine.** `internal/trust/apply.go:22` is the refusal, in those words. Any trusted key may add or revoke any other; none may evict the anchor. So a compromised key can lock out every engineer and cannot lock out the owner of the machine.

**The station publishes the set it is verifying against.** `internal/wire/status.go:41-43` carries the digest, the serial and name-to-fingerprint pairs with revoked ones marked, and the bash station emits all three on every transition (`station/bash/station.sh:498-500`). The earlier version of this page cited `internal/wire/status.go:15-30` and said the struct had no field for it. That was true of those sixteen lines and false of the struct, which now runs to `internal/wire/status.go:45`.

**And the control plane has no code path that authors a change.** `internal/cloud/authorship_test.go:69` builds the push path and fails if the code that signs is reachable from it.

**So why conditional rather than holds, and this is the part not to skip.** The mechanism makes an unauthorised key **detectable**. It does not make one impossible to add unnoticed by somebody who never looks. The published set is worth exactly what reading it is worth, and nothing on our side can make anybody read it. That is the named condition in the row.

**The third criterion landed partly, and the page is not going to round it up.** A change is not announced as a per-key audit event. What a reader gets is a serial that advances and a member list that differs from the one they saw last, so a change is found by comparison rather than delivered as a notification. That is better than configuration drift and it is not an audit event, and anybody building a control on top of it should build the comparison.

**A second limit, which is about time rather than attention.** Revocation on a beacon is eventual: the station learns of it on its next poll. "Revoked" reading as "instant" is the kind of assumption that gets discovered during an incident.

### The signature covers which step is named, not what it contains

Inside the signature: the estate, station, direction, sequence number, kind and recipient fingerprint (`internal/seal/seal.go:158-165`), plus the whole request document (`internal/wire/request.go:32-40`) - version, id, step, env, cancel, stop, note. The signing itself is `internal/seal/seal.go:261-262`.

Not inside it: the **content of the step**. `internal/wire/request.go:35` carries `Step` as "a registered name, or a path", and the file that name resolves to lives on the station.

Not closed by the published payload digest either. `station/bash/station.sh:90-93` hashes the station's runner and capture library, and `station/bash/station.sh:86-89` says why the steps are deliberately excluded: "those are SUPPOSED to differ per branch, and including them would make the digest change for the ordinary reason and stop meaning anything." That is a good reason, and its consequence is that **the digest a station publishes attests to its runner, not to what it will run.**

There is also no expiry, and its absence is argued for rather than overlooked: `internal/seal/seal.go:148-157` records that a signed timestamp was tried and removed, because a recipient cannot reconstruct a clock it did not read.

So the accurate sentence:

> A request is signed by its author and binds the estate, the station, the direction, the sequence number and the request document, which names the step to run and the environment to run it with. It does not bind the contents of that step.

### A timestamp gap, and what it does not prove

The mechanism is good. `station/bash/caplib.sh:385-393` stamps each line in a bash read loop as it arrives (`station/bash/caplib.sh:388`), before the ANSI strip and before redaction. A bash `read` is line-buffered by definition, so the stamp is taken when the line is produced and later buffering cannot change what time it claims.

**It establishes when the capture harness saw the line. It does not establish:**

- that the output was **produced** then. A command that buffers its own output emits ten minutes of work in one burst
- that the record is **complete**
- that the record is **authentic after capture**. On the relay transport the log is sealed for transmission, which signs the station sending it. On git, object store, Azure Blob, file share and bundle, nothing signs it at all

So: "a 4m12s gap in output", never "stalled for 4m12s".

### Only one transport seals

`station/bash/transports/relay.sh` is the only transport that invokes the sealing tool. `git.sh`, `objstore.sh`, `blob.sh`, `bundle.sh` and `share.sh` carry the request and status documents as plaintext, and their trust rests on the access control of the store underneath. `_git_verify` at `station/bash/transports/git.sh:598` checks read reachability and write reachability and verifies no signature, because there is none.

That is a defensible design: on a git transport you already own the git host and its access control, and adding a second trust root would be the sort of clever that gets a design an unfavourable audit. But it means **every sentence beginning "a forged request dies on `ed25519.Verify`" carries "on the relay transport" or it is false on five transports out of six.**

## What this page does not cover

**The beam**, because the beam is not built. `site/content/transports.md:27-28` says it is "designed, and not yet a transport you can pick". The claims we intend to make about it - that Noise keys are ephemeral and held only at the two ends, that a forged frame fails authentication and the session drops - are intentions, and they are named here rather than in a row so nobody mistakes them for mechanisms.

**Anything about the broker's internals**, because the broker is not built either.

## When this page changes

Each **owed** row changes when its mechanism ships, and the row will then name the test rather than the intention. Each **retired** row stays on the page permanently, because the useful part is knowing which sentences were wrong and why.

The source of this page is public and takes pull requests.
