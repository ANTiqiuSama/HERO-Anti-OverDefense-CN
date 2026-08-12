# Case catalogue

Observed behaviours, described. Each entry says what was asked, what the agent
did instead, why that is over-defense, and what the proportionate version looks
like.

**On sourcing.** Entries come from a public community discussion of
coding-agent over-defense and from maintenance work on the ARIS repositories.
Everything is paraphrased into a behaviour description: no usernames, no quoted
complaints, no attribution. What matters is the shape, not who hit it.

**On what belongs here.** A case needs a shape someone else can recognise in
their own logs. "The model was annoying" is not a case. "Asked to diff two
spreadsheets, it hashed every row" is.

IDs are `AOD-<family>-<nnn>`. Families are defined in [`../RULES.md`](../RULES.md).

Long-running research can combine several families at once. See the anonymized
cross-family case: [实验治理吞掉了实验本身](research-governance-loop.md).

---

## How to use this

**Not by pasting it into your agent's config.** This file is roughly eight times
the size of the block, and an agent carrying eighteen worked examples of *this is
over-defense* will start matching real findings against them and dismissing the
ones that rhyme — which is the failure the contract exists to prevent. The block
already carries six one-line examples for calibration; that is as much of this
catalogue as belongs in a file loaded on every turn.

**Use it the other way round: for the argument after the fact.** When your agent
insists some hardening is necessary and you think it is not, quote the entry —
the ID, or a link to the line — and ask it to say how its proposal differs from
that case. That question has an answer. "It's just being careful" does not.

The IDs exist for exactly this. `AOD-PROC-003` is quicker to type than a paragraph
about why re-running a passing suite proves nothing, and it points at a
description you did not write in the middle of an argument.

**Read the counterexamples before you use any of this as ammunition.**
`AOD-HASH-003`, `AOD-EDGE-004`, `AOD-PROC-004` and `AOD-PROC-005` are here because a flat
reading of the rules would have suppressed something real: a hash that paid for
itself, a rare-looking bug the project's own docs produced, a smoke run that was
owed, and a broad regression run whose whole point was not knowing what would
break. If your agent cites one of *those* back at you, it may well be right.

---

## H — Hashing

### AOD-HASH-001 · Row-wise hashing to compare two spreadsheets
**Asked:** diff two Excel files and merge the differences into one.
**Did:** computed a hash for every row in both files, then compared hashes.
**Why it is over-defense:** ordinary cell comparison already answers the
question. The hash replaced nothing more expensive; it added a full pass over
both files plus the comparison that still had to happen. A five-minute task took
about half an hour.
**Proportionate:** compare the cells. Reach for a digest only when it lets you
*skip* work — see AOD-HASH-003.

### AOD-HASH-002 · Checksum files scattered through the working tree
**Asked:** ordinary development work.
**Did:** wrote SHA256 files across the machine as a side effect, with nothing
reading them afterwards.
**Why it is over-defense:** a digest nobody consumes is not verification, it is
residue. It also creates the impression that integrity is being checked when
nothing checks it.
**Proportionate:** if no code path reads a digest, do not write one.

### AOD-HASH-003 · The legitimate case, recorded so the rule stays honest
**Asked:** repeated work over the same large files across a long session.
**Did:** compared a file's digest against the previous one to decide whether it
needed re-reading, skipping unchanged files.
**Why this is NOT over-defense:** the hash replaced a materially more expensive
operation — pulling a large unchanged file back into context — and its result
changed the next action. This is the counterexample that makes a flat "no hashes"
rule wrong.

---

## E — Edge cases

### AOD-EDGE-001 · A whole day on account security for a toy app
**Asked:** a small prediction-market-style demo application.
**Did:** spent the session on administrator-account security and validation.
Feature code: none.
**Why it is over-defense:** the application had no administrators, no users and
no deployment. The defended threat did not exist.
**Proportionate:** build the feature. Security work begins when there is
something and someone to protect.

### AOD-EDGE-002 · Adversarial hardening for an offline test platform
**Asked:** an offline testing platform.
**Did:** added tamper-proofing, checksums, adversarial defense, evidence trails
and gating. The main feature finished early; the remaining time went into
boundary cases.
**Why it is over-defense:** the requirement named the setting, and nothing in the
project described an adversary, untrusted input, or a second party. "Offline"
alone would not establish that — local inputs still have a provenance — but the
agent did not reason about provenance either; it added the defenses from the
category, not from anything in this project.
**Proportionate:** ask what actually reaches this code and from whom. If the
answer is "the person running it", say so and build the feature.

### AOD-EDGE-003 · Version-migration planning for a local edit
**Asked:** modify some code locally.
**Did:** raised version numbering and old-to-new migration paths.
**Why it is over-defense:** there was no release, no consumer and no old
version in the field.
**Proportionate:** migration planning starts when something has shipped.

### AOD-EDGE-004 · The counterexample: a rare-looking case that was real
**Asked (ARIS, 2026-08):** review an experiment scheduler.
**Found:** a phase dependency written as a bare string instead of a list was
iterated character by character, so a dependent stage never became ready and the
scheduler never terminated.
**Why this is NOT over-defense:** it sounds like a rare input-shape edge case.
But the project's own documentation example used the bare-string form, so every
user following the docs produced it. *Here* is the operative word in the E rule.

---

## R — Rubrics

### AOD-PROC-001 · Overnight adversarial auditing with no progress
**Asked:** implement a feature.
**Did:** produced patches, then audited them adversarially in a loop. A full
night later the audit was still running and the requirement was untouched; the
token budget went to the audit.
**Why it is over-defense:** re-verification without a live uncertainty is not
verification. Nothing the later rounds found changed what happened next.
**Proportionate:** review once when the work is done, then test it by hand.

### AOD-PROC-002 · A code review that nothing can pass
**Asked:** review code.
**Did:** returned a failing verdict on every submission regardless of author or
model, always finding something to object to.
**Why it is over-defense:** a reviewer that never passes carries no information.
The verdict stops being a signal and becomes a fixed output.
**Proportionate:** a review that can pass. Say plainly when something is correct.

### AOD-PROC-003 · Re-running everything when nothing new is in doubt
**Asked:** change one line in a leaf module with no other callers.
**Did:** ran the entire suite, then re-ran it again after an unrelated edit
elsewhere.
**Why it is over-defense:** line count is not the finding — blast radius is. A
one-line change to a shared interface deserves a broad run (see AOD-PROC-005). This
one could not reach anything the suite covers beyond its own module, and the
second run re-checked code that had not moved since it last passed.
**Proportionate:** scope the run to what the change can reach. If you cannot say
what the extra coverage could still surprise you with, it is not adding
information.

### AOD-PROC-004 · The counterexample: a smoke test that was owed
**Asked (ARIS, 2026-08):** nothing — this is a defect that shipped.
**Found:** a scheduler judged every healthy job dead at its first sixty-second
poll and killed its session, because a pattern intended to match a tab matched a
literal backslash-t instead. It then looped forever. The file carried detailed
prose about its state machine.
**Why this is NOT over-testing:** the code had never been run once end to end. A
single smoke run would have exposed it in sixty seconds.

### AOD-PROC-005 · The counterexample: a broad run whose point is not knowing
**Asked:** change a shared serialization format used by several components.
**Did:** ran the full cross-component suite, unable to say in advance which
component would break.
**Why this is NOT over-testing:** the rule asks what specific failure a run would
detect. Read too literally it forbids exactly the runs whose purpose is to find
out *which* thing broke. The live uncertainty here is real and bounded — "some
consumer of this format may not survive it" — and the outcome changes the next
action: fix or roll back. Naming a bounded class of breakage is a good enough
answer; only "it might catch something" is not.
**Proportionate:** this is proportionate. Scope the run to the consumers of the
thing you changed, not to everything.

---

## O — Overbuild

### AOD-BUILD-001 · Guards guarding guards
**Asked:** ordinary implementation work.
**Did:** added a protective layer, then a layer protecting that layer, and
continued.
**Why it is over-defense:** each layer's justification is the previous layer,
not the requirement. There is no natural stopping point, which is the tell.
**Proportionate:** every layer should trace to something that was asked for.

### AOD-BUILD-002 · Defensive code outweighing the feature
**Asked:** implement a feature.
**Did:** produced more defensive code than implementation.
**Why it is over-defense:** not because of the ratio — a parser or a payment path
can legitimately be mostly validation. It is over-defense when you go through the
defensive code line by line and cannot name, for each guard, the input that
reaches it. Here most of them guarded states the caller could not construct.
**Proportionate:** keep the guards whose triggering input you can name; delete the
rest. A ratio is a prompt to go and look, never the verdict — using it as the
verdict is itself AOD-PROC.

### AOD-BUILD-003 · Session and salt machinery for a trivial call
**Asked (by a systems-security engineer):** tighten the interfaces of their own
system.
**Did:** wrapped things in large, hard-to-maintain structures and opened
assorted sessions with different salts for one simple call.
**Why it is over-defense:** notable because the reporter works in security and
still judged it excessive. The complaint is not "security is unnecessary", it is
"this is not the security this needs".
**Proportionate:** harden the interface that was named. They reported switching
to an earlier model and the problem going away — see the ceiling note in RULES.

### AOD-BUILD-004 · Sub-agent fan-out where one pass would do
**Asked:** a single task.
**Did:** started a multi-stage workflow with several sub-agents, each re-reading
the same code, running over an hour.
**Why it is over-defense:** parallel re-reading is not more rigour, it is the
same look repeated. Cost scales with agents; information does not.
**Proportionate:** fan out when the shards genuinely see different things.

---

## Sibling: over-correction (not a core family)

Recorded because it shows up in the same reports and fits the same hypothesised
root — optimising for not being blamed — but it is a different symptom, and none
of the four letters covers it. See RULES.

### SIB-001 · Abandoning a direction on one objection
**Observed:** raising a single problem with a plan causes the agent to apologise
extensively and reverse the whole approach rather than address the problem. A
small directional correction produces a large overshoot.
**Why it matters:** the user loses the ability to give partial feedback. Every
comment becomes all-or-nothing.
**Proportionate:** fix what was raised; keep what was not challenged.

### SIB-002 · Answering the prompt's framing instead of the question
**Observed:** when a prompt mentions an emphasis or a concern, the output
foregrounds that emphasis rather than answering on the merits — including a
recognisable "not X, but Y" construction used to restate rather than inform.
**Why it matters:** the agent reflects the question back with confidence, so the
user cannot tell agreement from analysis.
**Proportionate:** answer the question; mention the framing only if it is wrong.

---

## Contributing

A useful entry has: what was asked, what the agent did, why that is
disproportionate, and what the proportionate version looks like. The last field
is the one that matters — a catalogue of complaints without remedies is a wall,
not a resource.

Counterexamples are wanted too. `AOD-HASH-003`, `AOD-EDGE-004`, `AOD-PROC-004` and
`AOD-PROC-005` exist because a flat reading of the rules would have been wrong, and
the catalogue is where that gets caught.
