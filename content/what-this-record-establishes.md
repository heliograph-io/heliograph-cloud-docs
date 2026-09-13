# What a heliograph record establishes, and the three things it does not

Somebody is going to put one of these logs in front of a client, an auditor or a regulator and say "this is what happened".

This page exists so that what you say next is exactly right. It is short, and the three sections headed "does not" are the ones worth reading twice.

## What it does establish

**A UTC timestamp on every captured line, applied when the line was produced.**

The capture wrapper pipes the command through a bash read loop that stamps each line as it arrives (`station/bash/caplib.sh:385-393`, the loop at `station/bash/caplib.sh:388`), and it does that **before** colour codes are stripped and before credentials are masked. A bash `read` is line-buffered by definition, so the stamp is taken when the line is produced. Later stages can delay when a line is displayed; they cannot change what time it claims.

That ordering is not an accident and it was not free. The stamp used to be applied last, after `sed`, which made the whole property depend on `sed -u`: a `sed` without it buffers, an entire block arrives at the stamping loop at once, and every line in that block carries the same time **while the log still reads perfectly**. `tests/test-start.sh:73` records the correction.

**A non-truncated capture.** The whole of what the command printed, not a tail, not a summary.

**Provenance about the run**, in a header the capture writes: which commit, which host, which user, when it started (`station/bash/caplib.sh:303-324`), and a footer with the exit code and the finish time.

**A published state for the run**, distinct from the log. `internal/wire/status.go:15-30` carries the state, the request id, the step, the host, the branch, the start and finish times, the exit code, the path to the log, and a digest of the station payload that produced it.

**Signed sequence numbers per direction**, on the relay transport, so a gap in the series is visible to the recipient and a relay in the middle cannot lie about the numbering (`internal/seal/seal.go:184`).

That is a strong record. It is stronger than a pasted terminal scrollback by a distance, which is the thing it usually replaces.

## What it does not establish

### One: a gap in the timestamps does not prove a hang

It proves **silence at the point of capture**. That is all, and the difference is not pedantry.

A command that buffers its own output emits ten minutes of work in one burst. The gap in the log is real, the process was working perfectly throughout, and any sentence of the form "it stalled for four minutes" is an inference presented as an observation.

So the correct sentence is **"a 4m12s gap in output"**, and never "stalled for 4m12s". If one of our interfaces says otherwise, that is a defect and we would like to know.

A gap also does not distinguish between: a buffering command, a slow command, a hung command, a machine that went to sleep, and a capture that was interrupted. Deciding which requires evidence the log does not contain.

### Two: it is not yet an audit trail, and the difference is provenance

An audit record needs things a timestamped text file does not have. Some exist today and some do not, and the honest position is to say which:

| | today |
|---|---|
| **sequence numbers**, so a gap is detectable | **yes**, per direction, signed by the sender on the relay transport (`internal/seal/seal.go:184`), so a relay cannot lie about them |
| **explicit run completion**, distinguishing "ended" from "we stopped hearing" | **yes.** `internal/wire/status.go:119-125` knows the difference, and `undelivered` is its sharpest case: the run finished, the log is complete, and the transport would not carry it |
| **the payload that ran**, so two stations can be compared | **partly.** A digest is published (`station/bash/station.sh:90-93`) and it covers the runner and the capture library, deliberately **not** the steps (`station/bash/station.sh:86-89`) |
| **monotonic run timing** alongside wall clock | **no.** A station whose clock steps backwards produces a log that reads as though time reversed, and nothing in the record would show it |
| **which transport, which credential, which message id** | **no**, not carried in the record |
| **who authored the request**, by name | **yes, now.** `internal/wire/status.go:44` carries `By`, emitted at `station/bash/station.sh:912`: the name, from the station's own trusted set, of the key that signed the request this run came from |
| **who may command the station at all** | **yes, now.** `internal/wire/status.go:41-43` publishes the trusted set's digest, its serial, and name-to-fingerprint pairs with revoked ones marked |
| **whether the station may change anything** | **yes, now.** `internal/wire/status.go:30` carries `Actions` as `allowed` or `refused`, emitted at `station/bash/station.sh:488` |

**The three "yes, now" rows are new and they change what you may say in a room with a lawyer in it.**

This page used to say that "Alice sent this command" was not established by anything in the record, because there was one identity per estate. **That is no longer the case.** A station verifies against a trusted set rather than a single key, the set names its members, and the station publishes both the set and the name of the member whose key signed the request a given run came from.

So the defensible sentence has got stronger, and it is still worth writing carefully:

> This request was signed by the key registered to *Alice* in the station's trusted set, and the station published that set alongside the run.

**Not** "Alice ran this command". What is established is that a key **registered to Alice** signed it. Whether Alice was holding that key at the time is a question about key custody on Alice's machine, and nothing in the record can answer it. That gap is small, it is real, and it is the one somebody competent will ask about.

**And the set is worth quoting alongside the log**, because it is what makes the name meaningful. A `by:` line naming Alice, without the set that says which key Alice is, establishes less than it appears to.

### Three: `read-only` does not prove nothing changed

A step declares its own mode, in a comment in its own file, and the runner reads that declaration from the file it is about to execute (`station/bash/run.sh:260`). A step with no declaration or an unrecognised one refuses outright (`station/bash/run.sh:270-288`). A step declaring `action` additionally needs `CONFIRM=yes` in the request and a station started with `--allow-actions` (`station/bash/station.sh:192`), and a station without that flag refuses and publishes the refusal (`station/bash/station.sh:888`).

All of that is real, it fails closed, and it is worth having.

**It is a declaration and not a sandbox**, and the runner's own source says so at `station/bash/run.sh:253-254`:

> What this does NOT do, and the documentation says so too: stop an author declaring read-only and then writing `rm -rf`. Nothing in a shell runner can.

**So the gate catches the mistake, not the adversary.** A wrong header pasted onto a destructive step is caught. An author who writes `read-only` at the top of a file that changes things is not.

**And the blast radius is wider than the step's own files.** It is the account the station runs as, plus everything that account can reach: credentials on disk, sudo rights, sockets, downstream services. The toolkit refuses to run as root for exactly that reason (`station/bash/caplib.sh:261-288`): "this toolkit has no credentials of its own, so the account it runs as is the whole blast radius. As root that is the machine."

The accurate sentence for a report: **"the step declared itself read-only and the station accepted that declaration"**, not "the step could not have changed anything".

## And one more, because it comes up

### The record is not provably complete

Sequence gaps make loss **detectable**, which is the most that is honestly achievable and is genuinely useful. It is not the same as completeness, for three independent reasons:

1. **the operator controls what is pushed.** A station that finished a run and could not deliver the log publishes `undelivered` rather than a log. That is a state you can act on, and it is not a log
2. **on a git transport, history can be rewritten.** The transport repository is an ordinary git repository with ordinary git properties
3. **the relay deletes on collection and expires after seven days** (`heliograph-relay/relay.go:213`, TTL at `heliograph-relay/relay.go:119`). A collector may now take a lease and acknowledge afterwards (`heliograph-relay/lease.go:152`), which removes the case where a collector crashed between reading and writing. Anything not collected and kept is still gone

**So completeness state travels inside every result and every export**, rather than living on the page you downloaded it from. A document that leaves our console carries what it is missing, in the document, because an export circulated stripped of its caveats is exactly the failure this page exists to prevent.

## How to describe one of these records accurately

If you are writing this into a report, these forms are defensible:

| instead of | write |
|---|---|
| "the process stalled for 4m12s" | "there was a 4m12s gap in captured output" |
| "the step could not have changed anything" | "the step declared itself read-only and the station accepted that declaration" |
| "a complete record of the session" | "a captured record whose gaps would be detectable" |
| "Alice ran this command" | "this request was signed by the key registered to Alice in the station's trusted set" |
| "the log proves the command ran at 02:14" | "the capture harness recorded this line at 02:14 UTC" |
| "this is an audit trail" | "this is a captured record with signed sequencing and published run state" |

None of those is weaker in a way that matters. Each is a statement you can defend when somebody competent asks the follow-up question, and the first version is not.

## Why this page exists at all

Because the alternative is that somebody else is precise about this first, in a meeting, after a claim has been made.

A record that establishes less than people assume is not a weakness of the product. A record whose limits are discovered by an opponent is.
