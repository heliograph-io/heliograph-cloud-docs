# What heliograph cloud can do, and what it cannot

The relay page sets the standard this page has to meet. It leads with what the thing *can* do, and then says what it cannot, and every line of it is checkable against source you can read.

The cloud has a harder story to tell, so it goes first and in plainer words.

## It reads your logs in plaintext

**That is the trade. It is at the top of this page rather than in a footnote, because it is the thing you are actually deciding about.**

The archive has to read your logs. Searching them, alerting on them and answering a question across forty estates are the reasons the service is worth paying for, and none of them work on ciphertext. There is no setting that turns this off, because a version that could not read your logs would be a different product.

Alongside that, it can:

- see every estate, station, step, timing and exit code
- see who authored and approved what
- refuse to deliver, or delay delivery, on the broker
- with the beam, when the beam exists, see connection metadata: class, destination, peer, times, byte counts

## What bounds it, honestly weighted

Three things, weakest first, and none of them eliminates the point above.

**`REDACT` is on by default, station-side, before delivery.** `station/bash/caplib.sh:205-220` masks about a dozen credential shapes - passwords, bearer tokens, credentials in URLs, GitHub and GitLab and Slack and AWS and OpenAI key formats, JWTs, private key headers - and it runs before the log is ever sent, so the masked value is what we receive rather than what we store. It is on unless you turn it off: `station/bash/caplib.sh:206` returns early only when `REDACT` is explicitly `0`.

**It is a safety net, not a guarantee, and the project's own code calls it "best-effort secret masking"** (`station/bash/caplib.sh:26`). It catches credential *shapes*. It does not catch a hostname, a database row, a customer name, an internal URL, or the contents of a config file your step printed. The corpus it is tested against is `tests/test-redact.sh` and `tests/test-redaction-corpus.sh`, both public.

**Self-hosting the relay removes the relay from the question. It does not remove the archive**, and the archive is the proprietary part. Saying "it is self-hostable" and stopping there would let that phrase do more work than it can carry. If a third party reading your captured logs is outside your threat model, the answer is a self-hosted relay and no archive, and this page would rather say that than argue with you.

**Customer-managed keys** would let you hold the receiving key and have us store ciphertext we cannot read. It kills search and alerting, so it is a different product rather than a setting. **It is not committed** and there is no date.

## What it cannot do, and what each one rests on

Each claim below names the mechanism. Where the mechanism is a discipline rather than code, the row says so, because a constraint you have to trust us about is worth less than one you can check and we would rather you knew which you were getting.

### It cannot forge a request a station will accept, on the relay transport

It holds no Ed25519 signing key. `Seal` signs with the private half at `internal/seal/seal.go:262`, and a forgery dies at `internal/seal/seal.go:355` on `ed25519.Verify`. Before that, the station constant-time compares the claimed sender against the one it expects (`internal/seal/seal.go:322-324`). A failure is a hard refusal with no warn-only mode, and the function's own comment says why: "There is no mode in which an unverifiable message is acted on, because the whole point is that a hostile relay must not be able to cause a run" (`internal/seal/seal.go:302-307`).

The refusal is published as a status, so you find out.

**Proved by** `TestARelayCannotForgeARequest` (`internal/seal/seal_test.go:71`), `TestAMessageClaimingAnotherSenderIsRefused` (`internal/seal/seal_test.go:161`) and `TestGoldenVectors` (`internal/seal/vectors_test.go:141`), which pins the wire bytes so a second implementation cannot quietly agree with only itself.

**Scope, and it matters.** This is a property of the **relay** transport. Only `station/bash/transports/relay.sh` seals: on git, object store, Azure Blob, file share and bundle the request travels as plaintext and what protects it is the access control of the store underneath, which you own and can check. `_git_verify` at `station/bash/transports/git.sh:598` checks read and write reachability and verifies no signature, because on that transport there is none to verify.

### It cannot add a key to your station's trusted set, and you can check that yourself

**This is the constraint that makes the claim above mean anything.** Signature verification only means a station trusts whatever key it was told to trust. If we could add a key we controlled, we could sign legitimately, your station would verify happily, and nothing would have been stolen or forged.

**Until September 2026 this section said that was a promise rather than a property.** It is now a mechanism, in four parts.

**A change to the trusted set is itself a signed document**, verified against the set as it stands. An unauthenticated change is indistinguishable from an unauthenticated request, and is refused the same way with a published reason.

**The anchor is changeable only on the machine.** `internal/trust/apply.go:22` is the refusal: `"trust: the anchor is changeable only on the machine"`. Any trusted key may add or revoke any other, which is what makes it possible to offboard an engineer across forty estates without visiting any of them. **None may evict the anchor.** So a compromised key can lock out every engineer and cannot lock out the owner of the machine, and recovery is a signed request from whoever holds the anchor rather than a site visit.

**Your station publishes the set it is verifying against.** `internal/wire/status.go:41-43` carries three fields - the digest, how many changes have been applied, and name-to-fingerprint pairs with revoked ones marked - and the bash station emits all three on every transition (`station/bash/station.sh:498-500`). The field's own comment says what it is for:

> This is what lets an estate owner audit the trusted set from their own transport, with the CLI, without asking us - and a key appearing that nobody authorised is then independently detectable rather than something they have to trust us to notice.

**And the control plane has no code path that authors a change.** `internal/cloud/authorship_test.go:69` builds the push path and fails if the code that signs is reachable from it. `internal/trust/set.go:90` keeps the member list unexported with a single mutator, so the compiler enforces the other half rather than a reviewer.

So the sentence:

> heliograph cloud cannot add a key to your station's trusted set. A change is a signed document your station verifies, the anchor can only be changed on the machine itself, and your station publishes the set it is verifying against - so you can audit who may command your estate from your own transport, with the CLI, without asking us.

**Two limits, and they are the reason this is not the end of the subject.**

**It makes an unauthorised key detectable, not impossible to add unnoticed by somebody who never looks.** The published set is only worth what reading it is worth. If nobody ever compares it against what they expect, the mechanism has done its half and nothing has done the other half. That is strictly better than the previous position, where it was not detectable at all, and it is not the same as prevented.

**Revocation on a beacon is eventual.** Your station learns of it on its next poll. It is immediate only if something terminates an open session, which is separate work. "Revoked" reading as "instant" is the kind of assumption that gets discovered during an incident, so the word here is **eventual**.

### It cannot hold your transport hostage to our control plane, for more than 15 minutes

**A hosted service that authorises every operation turns our outage into your outage, at exactly the moment you needed the transport.** heliograph exists to reach machines when things are broken. That failure mode is the opposite of the proposition, so it is designed against rather than hoped about.

Authority is a **signed lease** the relay validates locally, not a boolean it fetches per call. A control-plane outage therefore leaves existing leases working until they expire, and `TestAControlPlaneOutageLeavesExistingLeasesWorking` (`heliograph-relay/authlease_test.go:149`) proves it by killing the control plane. `TestALeaseIsValidatedWithNoNetworkCall` (`heliograph-relay/authlease_test.go:86`) counts the calls the inner authoriser receives during validation and gets **zero**.

**New enrolment and any privilege increase still fail closed during that outage**, and distinguishably: both answer `503 authoriser-unavailable` rather than `401`, and `TestEnrolmentAndPrivilegeIncreaseRefuseDuringAnOutage` (`heliograph-relay/authlease_test.go:181`) fails if either answers `401`. A capability increase is not something to hand out because we cannot reach our own database.

**The maximum revocation delay is 15 minutes, and it is a number rather than a word.** `MaxAuthorityLife` at `heliograph-relay/authlease.go:56`, published in the relay's own README, and `TestTheMaximumRevocationDelayIsTheNumberWePublish` revokes access and steps a clock until it stops, reporting `access stopped 15m0s after revocation; the published maximum is 15m0s`.

**The honest limit is the same trade seen from the other side.** A lease that runs to its expiry after revocation means revoked access persists for up to 15 minutes. That is deliberate: the alternative is a shorter lease and a harder dependency on us being up. There is no TTL that removes the trade-off, and we would rather publish the number than describe it as "promptly".

Three behaviours are specified rather than left to the TTL, each with a test: a poll already open ends when its lease expires, a poll already open ends when its authority is revoked, and **revocation does not recall a message already delivered** - which is the honest limit of the three and is named as such.

### It cannot read a message in transit on the relay

The relay stores and forwards opaque bytes. `heliograph-relay/relay.go:1-20` is the argument and `heliograph-relay/relay.go:57` is the type: `Body []byte` with the comment "ciphertext. Opaque here, always". Content is encrypted under a key derived per message from an ephemeral X25519 key (`internal/seal/seal.go:227`) held only at the two ends.

**One sentence in that package comment was withdrawn in September 2026 and we are not going to let it pass quietly**, because it was offered as a reason to trust the relay and somebody may have approved it on the strength of the sentence.

It used to open **"THERE IS NO CRYPTOGRAPHY IN THIS PACKAGE"**. It does not any more. The relay now verifies authorisation lease signatures with Ed25519, in both implementations, because a lease minted by a control plane and handed to a relay that has never seen it can only be honoured by checking a signature. The alternative was trusting unsigned leases, which means trusting forged ones, or refusing every lease, which would have removed the outage protection in the section above and made the feature theatre.

**The proxy narrowed. The claim it stood for did not, and that is the part to check rather than take on trust:**

- **a public key is not a secret.** An attacker who takes everything this relay holds gets a key that checks signatures and makes none
- **HMAC-SHA256 was rejected**, though it would have passed the old rule as written, because it is symmetric: a relay able to verify would be able to **mint** any lease it liked. That ends the claim rather than narrowing the sentence
- **the narrowing is held in place by the linker, not by a comment.** `TestTheRelayBinaryCannotSignALease` builds the relay and reads its symbol table. Go links only reachable code, so a binary with no route to constructing an Ed25519 private key has no code path that could sign anything. A companion test builds the conformance harness, which does sign, and fails if the check cannot tell the difference
- `heliograph-relay/verify.go:11-44` carries the whole argument in the file that caused it, and `heliograph-relay/verify.go:51-57` fails closed on a malformed key, because a verifier that accepted everything because its key was misconfigured would accept a forged lease and the forger chooses the scope

**What it cost you:** you can no longer confirm the old sentence with one `grep`. The narrower rule is enforced rather than asserted, which is stronger, and it is less immediately checkable by somebody who is not going to run a test. The relay's README says the same thing in the same words.

**The sentence that is true today:** there is no private key in the relay, it cannot sign anything, and that is enforced by the linker.

**Proved by** `TestTheCiphertextRevealsNothing` (`internal/seal/seal_test.go:50`) and `TestTamperingWithTheCiphertextIsDetected` (`internal/seal/seal_test.go:143`).

**What this does not cover is the archive**, which is the first section of this page.

### It cannot replay a message, while your station keeps one file

The sequence number is inside the signature (`internal/seal/seal.go:184`), so a relay cannot lie about it, and the recipient refuses anything at or below the highest it has already accepted.

**The condition is that the recipient remembers.** The station keeps that state in a local file (`station/bash/transports/relay.sh:121-126`), written with a write-then-rename that takes the higher of each field so a stale writer cannot wind the counter back (`station/bash/transports/relay.sh:259-266`). The code states the failure before we do:

> Write-then-rename, because a truncated state file reads as sequence zero, and sequence zero accepts every replay the relay has ever seen.

`station/bash/transports/relay.sh:257-258`. Restore a station from a backup taken before the counter advanced, or lose that file, and replay protection is gone until it catches up.

### It cannot read a beam, and cannot inject into one

Noise keys are ephemeral and held only by the two ends, and a forged frame fails authentication and the session drops.

**Both of those are properties of a design rather than of code, because the beam is not built.** `site/content/transports.md:27-28` says so in the public docs: "It is designed, and not yet a transport you can pick." This section will name a mechanism and a test when there is one. Until then it is an intention, and it is listed here so you can hold us to it rather than discover later that it was assumed.

## The two-token asymmetry, and why a station token is weaker on purpose

If you run a station, it holds a credential. That credential is deliberately less capable than the one on your control node, and it is worth understanding why before you accept it.

| token | may | may not |
|---|---|---|
| **station** | read requests, write status and logs | **queue a request, even for its own station** |
| **control** | write requests, read logs | write status or logs as the station |

**And the scope is per station rather than per estate.** An earlier shape gave one credential per estate in both directions, which meant a station token taken from one client could read another's queues and write fabricated replies into them wherever a single account held several clients. Station-scoped authorisation is a launch dependency rather than hardening for later, and cross-station denial is proved in a test rather than forbidden by policy.

**The asymmetry exists because a station token sits on a machine nobody can reach and cannot be rotated quickly.** That is the entire situation heliograph is for: the machine is behind a boundary, the person who can reach it cannot debug it, and getting a new credential onto it costs a round trip through somebody's schedule.

So if a station token could queue a request, a stolen one would let an attacker send work to that station. The station would refuse it, because it would not carry the control signature. But it would fill the queue, and it would mean a credential sitting on the least defensible machine in the estate could cause traffic the control never sent. `heliograph-relay/server.go:126-146` is the enforcement, and its comment is the argument in full.

**Tokens are not the security boundary for content or execution.** Those are settled by signatures the relay cannot make, so a stolen token yields no plaintext and cannot cause a station to run anything.

**They are the boundary for four things, and one of them is sharper than it looks:**

| a stolen token lets somebody | and the cost is |
|---|---|
| **collect** a queue | ciphertext they cannot read, and **the legitimate collector never gets it**, because collecting deletes |
| **fill** a queue to its limit | the real sender is refused and delivery stops |
| **spend** whatever is being metered | denial of service, on your bill |
| **cross a tenant boundary**, wherever one authoriser serves several customers | one customer's routing keys reachable with another's credential |

**The first row is the one that matters for an evidence product.** An unleased collection deletes in the same breath as it returns (`heliograph-relay/relay.go:213`, delete at `heliograph-relay/relay.go:239`). So a stolen token causes **silent loss, not silent disclosure** - and on this transport the sender is often a station nobody can log into, holding the only copy of an hour-long capture. An earlier version of this wording said "a stolen token yields denial of service and metadata", which was written when a lost message cost a re-run. On a retained archive it can cost the evidence.

**Leased collection now exists, and it changes that row for the honest collector rather than for the thief.** A collector can take a lease, write the messages down durably, and acknowledge afterwards (`heliograph-relay/lease.go:152` `TakeLeased`, `heliograph-relay/lease.go:192` `Ack`); if it dies in between, the lease expires and the messages come back.

**It does not remove the deliberate case, and this page is not going to imply that it does.** `?lease=` is the client's choice and the relay cannot tell a thief from anyone else, so a thief simply does not lease. What leasing removes is the **accidental** loss, which is the common case. What it does not remove is the deliberate one, which needs the token not to be stolen. Treat a station token as a credential worth protecting even though it cannot read anything.

## What the broker retains, per field

"The broker never sees your data" would be a headline claim, so here is its limit in the same breath. A reader competent enough to matter would derive this list anyway, and concluding we had hidden it is worse than publishing it.

| field | why it is held | retention |
|---|---|---|
| account identifier, owner, billing records | to bill you | life of the account, then as tax law requires |
| station identity and the group it joined | to route and to show you a fleet | life of the station record |
| enrolment and pairing events, with timestamps | to show who added what, and when | life of the account |
| connection times and durations | quota, liveness, fleet view | **to be set. See the note below** |
| message counts and sizes | quota and metering | **to be set** |
| routing and policy decisions, including refusals | to answer "why was that denied" | **to be set** |
| source IP addresses | abuse control and rate limiting | **to be set** |
| run records: estate, station, step name, timing, exit code, state | the archive. This is the product | your plan's archive retention |
| log bodies, in plaintext | search and alerting. This is the product | your plan's archive retention |

**The retention periods are not filled in and this page will not invent them.** Publishing a number we have not built the deletion for would be a claim we could not meet, and an unenforced retention policy is worse than a stated absence of one. They are set before signup opens, enforced by a mechanism rather than by intention, and the same list appears in the terms so it is contractual rather than only documentary.

### What traffic analysis on that list can establish

Plainly, because implying it away insults the reader this product is for.

**It can establish:** that an investigation happened, and roughly when. Roughly how large the captured output was. Which machines were involved. How often a given estate is active, and the shape of that activity over time. Whether a step ran long or short. Whether a station went quiet.

Put together across an estate, that is enough to infer that something was wrong with a particular machine on a particular night, and roughly how much output looking at it produced.

**It cannot establish:** what the step did. What the log said. What the exit code meant. Any content of any message.

**Padding and timing obfuscation are not implemented**, and the public relay documentation already says so rather than implying otherwise: message sizes and timing are not hidden, padding was considered and rejected for now because it costs bandwidth on links that are often poor and the leak is coarse.

**If a third party holding connection metadata is outside your threat model, the answer is a self-hosted relay**, and this page would rather say that than argue with you. You keep every transport shape and every passenger. What you give up is the archive, the fleet view and the alerting, which is what the service is.

## Three things this product must never imply, and does not

These are collected here because they are the sentences most likely to be repeated by somebody who did not read the rest of the page.

### `read-only` is a declaration, not a sandbox

A step declares its own mode in a comment in its own file, and the runner reads it from the file it is about to execute (`station/bash/run.sh:260`). Missing or unrecognised refuses, fail closed (`station/bash/run.sh:270-288`). An `action` step additionally needs `CONFIRM=yes` in the request and a station started with `--allow-actions` (`station/bash/station.sh:192`, refusal at `station/bash/station.sh:888`).

That gate is real and it fails closed. It is also honest about itself, in the runner's own source at `station/bash/run.sh:253-254`:

> What this does NOT do, and the documentation says so too: stop an author declaring read-only and then writing `rm -rf`. Nothing in a shell runner can.

**So the gate catches the mistake, not the adversary.** A wrong header pasted onto a destructive step is caught. An author who declares `read-only` and means otherwise is not.

**And the blast radius is wider than the account's own files.** It is the account plus everything the account can reach: credentials on disk, sudo rights, sockets, downstream services. The toolkit refuses to run as root for exactly this reason (`station/bash/caplib.sh:261-288`) - "this toolkit has no credentials of its own, so the account it runs as is the whole blast radius".

### A timestamp gap proves silence at the point of capture

Not a hang. Not a stall. Not a stuck process.

The mechanism is unusually good: `station/bash/caplib.sh:385-393` stamps each line in a bash read loop as it arrives (`station/bash/caplib.sh:388`), **before** the ANSI strip and before redaction. A bash `read` is line-buffered by definition, so the stamp is taken when the line is *produced*, and later buffering can delay when a line is displayed but cannot change what time it claims. That ordering was a correction: it used to be applied last, which made the property depend on `sed -u`, and a `sed` without it buffers a whole block so every line in it carries the same time while the log still reads perfectly.

So a gap is a real observation and "it hung" is an inference. A command that buffers its own output produces a four-minute gap while working perfectly. Our console says "a 4m12s gap in output" and never "stalled for 4m12s", and if you find a string of ours that does the other thing, it is a defect.

### The archive is not provably complete

Sequence numbers are signed by the sender, so a gap in the series is **detectable**. That is the most that is honestly achievable and it is genuinely useful.

It is not completeness, for three independent reasons: the operator controls what is pushed and a station that never delivers a log produces `undelivered` rather than a log; on a git transport history can be rewritten; and the relay deletes on collection and expires after seven days, so anything not collected and kept is gone.

**Completeness state therefore travels inside every result and every export**, rather than on the page you downloaded it from, so a document cannot be circulated stripped of its caveats. [What a heliograph record establishes](what-this-record-establishes.md) is the longer version and it is the page to read before putting a log in front of an auditor.

## What is deployed, and how you check it

**Nothing is deployed yet**, so this section describes what you will be able to check rather than what you can check today. It is here because it is the question a security reviewer asks third, after "can you read my logs" and "can you run commands on my machines".

**What exists now.** Both relay implementations are **reproducibly built**, and CI proves it on every pull request rather than asserting it: each is built twice from two directories, the second from a copy with no `.git`, and any difference fails the build. You can run `edge/reproduce.sh` or `packaging/reproduce.sh` against a tag and arrive at the same hash we publish. `GET /health` reports the version serving and the SHA-256 of the artefact serving it (`heliograph-relay/server.go:44-56`), and the Go binary hashes its own executable at startup rather than being told what it is.

**The finding that produced it is worth knowing, because only somebody verifying a download would ever have hit it.** The same commit built inside and outside a git checkout produced different binaries: the toolchain stamps version information by default and omits it silently where there is no repository. A reproducibility claim would have been false for exactly the person who bothered to check.

**What does not exist, stated so you do not have to find out later:**

- **nothing is signed.** The release workflow signs checksums with Sigstore and has never run, because no tag has been pushed
- **nothing is deployed**, so no hash of a running service has ever been compared against a build
- **a Worker cannot read its own code.** Its reported hash is the number the deploy workflow computed from the source it deployed. That is a public log of a public workflow, and it is weaker than a binary that hashes itself

**So the sentence we are entitled to today** is "the bundle a tag produces is a number you can check, and the deployment says it is serving that number" - not "the relay is provably the source you read". When signing lands and something is actually deployed, this section says so and names the command.

## A binary on your machine, and exactly which shapes need one

An estate that forbids compiled code is a real constraint rather than a hypothetical, and the answer is more precise than "it is just bash".

| what you want | what it needs on the far side |
|---|---|
| git, file share, bundle, object store | **nothing compiled**, on either side |
| relay, with the PowerShell station | **nothing compiled.** The construction ships as source and is compiled at startup |
| relay, with the bash station | `heliograph-seal`, because the end-to-end encryption needs X25519, ChaCha20-Poly1305, Ed25519 and HKDF, and `curl` and coreutils do not do those |
| the beam, when it exists | its own component |

**"Beacon and flare need no binary" is not quite right**, and this page says the accurate version instead: the relay *is* a beacon, and its bash station does need `heliograph-seal`. An estate that permits no compiled code loses **the relay on a bash station and the beam. Two shapes, not the tool.**

The list is a file rather than a policy note: `station/FAR-SIDE-BINARIES` in the public repository is read by CI, and a program whose name appears under `station/` and is not on that list fails the build. Adding one requires that the feature genuinely cannot be done in bash, that the build is reproducible so you can check the bytes against the published checksum, and that the station verifies the binary by hash before running it.

## The threat model

Every claim on this page, with its mechanism and the condition under which it fails, in one table: [the cloud threat model](threat-model.md).

It is a short page and it is the one to read if you are evaluating this adversarially. It says which claims hold outright, which hold conditionally, which are owed, and which sentences we have stopped saying.

## Reporting something

If a claim on this page does not match the code, the source of this page is public and takes pull requests.

For a security weakness in heliograph itself, `SECURITY.md` in the public repository asks you to email rather than open a public issue. A finding is published once its fix has shipped, intact rather than summarised, on Project Zero's timeline: a fix in 7 days where there is a present exploitation path and material harm, 90 days otherwise, and the advisory 30 days after the fix either way. That 30-day gap belongs to the operator who has to patch, not to us.

**Hosted customers get no earlier notice than self-hosters.**
