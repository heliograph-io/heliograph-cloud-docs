# What each transport gives you, and the one thing a beacon estate does not get

heliograph moves documents across a gap using whatever the estate permits. The archive can receive from any of them. **What differs is not the evidence, it is how fresh it can be**, and one of the differences decides whether a whole feature works for you.

Read this before you choose, not after.

## The short version

| how the archive gets your runs | the archive and the history | the request that caused each run | tells you when a station goes quiet |
|---|---|---|---|
| **beacon**, pushed by your control node | yes | no, and see below | **no** |
| **git**, read by us over a read-only credential | yes | yes | yes |
| object store, Azure Blob, file share | yes | yes | yes |
| bundle, carried by hand | yes | yes | no |

## The beacon path, and the limit that belongs on this page rather than in a support ticket

A beacon estate reaches the archive by **push**: your control node collects what the relay is holding, and `heliograph push` sends it on. You get the archive and you get the history.

**You do not get gone-quiet alerting.** Nothing reaches the service while your control node is closed, so the service cannot tell a station that has stopped from a laptop that has been shut. You cannot alert on silence when your only source is a machine that also goes silent.

That is not a gap we intend to close on this path, because there is nothing to close it with. It is the clearest reason to run a **git transport alongside**, which is free, needs only a read-only deploy credential, and works with the laptop shut.

What the beacon path does give you, and it is most of the product:

- every run your control node has collected, stored byte for byte
- the full history, searchable, with the gap query
- sequence numbers, which are the strongest completeness statement any transport here carries: the relay assigns one per direction and the sender signs it, so a missing push shows up as a **gap** rather than as silence
- a credential that can upload and can do nothing else

### And the honest flip side of "we hold no keys"

On the beacon path the service holds **no key material whatsoever** - not the signing half, not the receiving half. Your control node opens everything on your own machine and forwards the result. So "cannot sign, therefore cannot cause a run" is stronger here than anywhere else in the product.

In the same breath, because it would be dishonest to write the first sentence without the second: **what we receive is plaintext**. An uploader forwarding already-decrypted spool contents needs no private key at all. "We hold no keys" is true, and it is a statement about what this service cannot *do*, not about what it cannot *see*.

## The git path

We read your transport repository ourselves, over a **read-only deploy credential**, on a schedule. Nothing on your side has to be running.

- **it works with the laptop shut**, which is what makes gone-quiet alerting possible at all
- **we see the request as well as the result.** Requests are plaintext in a git transport, so the archive holds what was asked and what came back. `station/bash/transports/relay.sh` is the only transport that seals, which you can check by grepping the transport directory for `heliograph-seal`: `git.sh`, `objstore.sh`, `blob.sh`, `bundle.sh` and `share.sh` do not appear. Over a beacon the archive holds only what came back, because a request is sealed to the station's receiving key
- **reading takes nothing from you.** Git history is still there after it has been read. The relay is the only transport that is a queue, which is why the service never reads that directly
- **a rewritten history is reported, not hidden.** A force-push or a prune loses runs the archive already recorded. Every run stores the commit it was read out of, and a run whose commit is no longer in your branch's ancestry is reported as a gap

What you are granting, said plainly: **read access to a repository whose whole purpose is to hold captured output.** That includes anything committed to it before you thought about us. The station's redaction is a safety net rather than a guarantee.

## Object store, Azure Blob and file share

Same shape as git. A read-only credential, a non-destructive read, and the request visible because it is plaintext in the store. These are for an estate whose security policy permits a storage account and nothing else, which is a real and common shape.

## Bundle

The most locked-down case: there is no network path at all and somebody carries a file. You upload it. You get the archive and the history, and no freshness of any kind, because freshness is not a thing a stick has.

## What none of them gives you

**Absence is not proof of absence.** Whoever operates the near side decides what is uploaded, so somebody can decline to push a run and the archive cannot know. Sequence gaps catch accidental loss and turn deliberate omission into a visible gap rather than into silence, which is the most that is honestly achievable.

Git has the same limit for a different reason: an ancestry report is what the history contains today, and a repository nobody has pointed us at leaves nothing to compare against.

An archive trusted to be complete when it is not is worse than no archive, because somebody will cite it to a regulator. So the archive says which of the three it is - `complete`, `unverified`, or `gap-detected` with a sentence naming what is missing - on every answer it gives.
