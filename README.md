# heliograph cloud documentation

The public documentation for **heliograph cloud**, the hosted service. In [`content/`](content/), under **CC BY 4.0**.

heliograph moves documents across a gap you cannot cross yourself: you can reach a machine's operator but not the machine. The protocol, the CLI and the station are at [dbhq-uk/heliograph](https://github.com/dbhq-uk/heliograph); the relay server is at [dbhq-uk/heliograph-relay](https://github.com/dbhq-uk/heliograph-relay). This repository is the prose about the service built beside them.

## The pages

| | |
|---|---|
| [What is open source, what is fair source, and what is not either](content/boundary.md) | which component carries which licence, what you can run yourself, and the two sentences being retired because they are false |
| [What heliograph cloud can do, and what it cannot](content/security.md) | the security page. It opens with the fact that the service reads your logs in plaintext, because that is the thing you are deciding about |
| [The threat model, in one table](content/threat-model.md) | every claim, the mechanism that enforces it, and the condition under which it fails. The page to read if you are evaluating this adversarially |
| [What a heliograph record establishes, and the three things it does not](content/what-this-record-establishes.md) | read this before you put one of these logs in front of a client, an auditor or a regulator |

**A fifth page is owed**, on what each transport gives you and the one thing a beacon estate does not get. It is written and it is not here yet, for a reason worth stating rather than leaving as a gap: its wording is held in place by a test in the private repository that reads the file, and publishing the page without first rebuilding that guard here would quietly remove the only thing stopping an overclaim that an adversarial review already caught once. The page arrives when the guard does.

## What this repository is not

**It is not the service.** heliograph cloud is proprietary and its code is not published. Neither is the broker. What is published is the relay, which is the component in the data path, and everything the CLI and the station do.

**It is not marketing.** No page here states a price, a tier, an allowance or a launch date. Where a number is not measured, the page says it is not measured rather than estimating it.

**It is not a promise about a service you can buy.** Nothing is deployed. Several pages describe behaviour that is built and not running, and each says so at the point where it matters rather than in a disclaimer nobody reads.

## Why the source of these pages is public when the product is not

The product is proprietary. The documentation is not, and that is deliberate.

Every comparable business examined keeps its documentation source public, including for closed-source commercial products. For this product it goes further than convention. The security page's whole argument is that each claim is checkable without trusting us, and a security page whose own source is private and takes no corrections is weaker for no gain. It is also the page most likely to be read adversarially.

So every claim about code on these pages carries a `path:line` into a public repository, or a named test, or it is downgraded until it does.

## Corrections are welcome, and that is the point

**If a claim here does not match the code, that is a defect.** Open an issue or a pull request. We would rather have it here than as a surprise in somebody's procurement review.

The most useful correction is the least dramatic one. **Line numbers move**, and a citation that has slid a few lines still resolves to *a* line, so it reads as correct while pointing at the wrong thing. Four of those were found and fixed on these pages the day before they were published, and one of them pointed a sentence about key substitution at a paragraph about pricing. If you follow a citation and it does not say what the page claims, that is worth reporting even when you are sure it is just drift.

For a security weakness in heliograph itself, `SECURITY.md` in the public repository asks you to email rather than open a public issue. A finding is published once its fix has shipped, intact rather than summarised, on [Project Zero's](https://projectzero.google/vulnerability-disclosure-policy.html) timeline: a fix in 7 days where there is a present exploitation path and material harm, 90 days otherwise, and the advisory 30 days after the fix either way. Hosted customers get no earlier notice than self-hosters.

[`CONTRIBUTING.md`](CONTRIBUTING.md) has the house style and the sign-off requirement.

## Licence

[CC BY 4.0](LICENSE). Reuse it, quote it, translate it, republish it. Attribution is the only condition.

The licence covers this documentation. It does not cover the software it describes, which carries its own terms per component - see [the boundary page](content/boundary.md).
