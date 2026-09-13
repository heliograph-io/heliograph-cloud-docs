# Contributing

This repository is prose about a product. The most valuable contribution is **a correction to a claim that does not match the code**, and the bar for making one is low on purpose.

## Sign off your commits

Every commit needs a `Signed-off-by` line. That is the [Developer Certificate of Origin](https://developercertificate.org/): you are stating that you wrote the change or have the right to submit it under this repository's licence.

```
git commit -s -m "your message"
```

A pull request with an unsigned commit fails the `DCO` check and names the commit. Fix it with `git rebase --signoff` and force-push your own branch.

**There is no contributor licence agreement**, and there will not be one. CC BY 4.0 has no copyleft, so nothing about the hosted service depends on collecting copyright assignments. The DCO records provenance, which is the part that actually matters.

## What a good correction looks like

**Cite, do not assert.** Anything claimed about code carries a full path and a line: `internal/seal/seal.go:355`, never a bare `seal.go:355`. If you have not opened the file, do not describe what is in it.

**Check the citation rather than trusting it**, including your own. The failure mode is not a broken link - it is a citation that was right when it was written. Line numbers move, a drifted citation still resolves to *a* line, and nothing about reading the page reveals it. Six citations on these pages had drifted the day before they were published and four of them landed on a neighbouring line about the same subject, which reads as correct.

So a correction that says "this line number is now wrong, here is the right one, here is what the line says today" is a good pull request even when the sentence above it was true.

**Never state an inference as an observation.** "A 4m12s gap in output", not "stalled for 4m12s" - a command that buffers its own output produces a gap while working perfectly. This rule is the subject of one of the pages and it applies to the pages themselves.

**Never write "cannot" where the truth is "does not currently".** If a mechanism enforces it, cite the mechanism. If a test proves it, name the test. Otherwise write what is true. Every **cannot** on these pages is meant to survive somebody checking it, and one that does not is the defect worth reporting most.

**Every decision carries its rejected alternative and what it cost.** A decision without its rejected options gets re-argued every quarter.

## House style

Shared with [heliograph](https://github.com/dbhq-uk/heliograph/blob/main/AGENTS.md) and the sibling repositories, so a reader moving between them does not have to notice which one they are in.

**British English. Plain hyphens, no em dashes and no en dashes. No trailing full stops on headings.** The hyphen rule is mechanical rather than aesthetic: em dashes arrive with pasted model output and word-processor autocorrect, and a repository that mixes them reads as assembled rather than written. CI fails a build containing one.

**Declarative and concrete.** Simple technical English, short sentences, plain words. Say what happened and what to do about it, and lead with the result.

**The name is lowercase**, including at the start of a sentence.

**Soft-wrap markdown inside these files.** Do not hard-wrap it in issues, pull requests or comments either: GitHub soft-wraps anyway and hard wrapping makes text awkward to edit in the web UI for no gain.

**No host names, environments, client names or real logs.** Fixture-style examples are fine. A redacted real one is not, because redaction fails quietly.

**Commit messages say what changed and what it cost** - the measurement, the failure it prevents, the thing that was tried and did not work. A title is a sentence stating the finding, not a noun phrase.

## What belongs here and what does not

**Here:** the hosted service's user contract, its security model, what a captured record establishes, what each transport gives you, the licence boundary, and change notices.

**Not here:** anything about the protocol, the CLI, the station payloads or the wire format, which belong in [heliograph](https://github.com/dbhq-uk/heliograph); anything about relay behaviour, its architecture or its deployment contract, which belong in [heliograph-relay](https://github.com/dbhq-uk/heliograph-relay).

The rule is that a decision lives with **the component whose observable behaviour changes**. Everything else links to it. Nothing copies it, because two copies become two versions of the truth and then they drift.

## Reporting a security weakness

`SECURITY.md` in the public heliograph repository asks you to email rather than open a public issue, and that applies to a weakness in the product. A claim on one of these pages being wrong is not that - open it here.

## Licence

By contributing you agree your work is licensed under [CC BY 4.0](LICENSE), the licence on this repository.
