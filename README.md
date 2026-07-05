# Define, Diagnose, Prove

*A structured troubleshooting method for support engineers.*

> If you can't define a problem, you can't understand it.
> If you can't understand it, you can't solve it.

This is the reference version of the method: full detail, real tables, and the quick-reference card for keeping open during a live call. For the narrative version with more context on where this came from, see the LinkedIn article *(link to be added here once published)*.

## Contents

1. [Why this method exists](#1-why-this-method-exists)
2. [The method at a glance](#2-the-method-at-a-glance)
3. [Two operating modes](#3-two-operating-modes)
4. [Phase 1: Triage](#4-phase-1-triage)
5. [Phase 2: Define the problem](#5-phase-2-define-the-problem)
6. [Phase 3: Scope the differences](#6-phase-3-scope-the-differences)
7. [Phase 4: Hypothesise wide](#7-phase-4-hypothesise-wide)
8. [Phase 5: Surface the assumptions](#8-phase-5-surface-the-assumptions)
9. [Phase 6: Prune the list](#9-phase-6-prune-the-list)
10. [Phase 7: Prove the cause, fix, verify](#10-phase-7-prove-the-cause-fix-verify)
11. [Phase 8: Record the RCA](#11-phase-8-record-the-rca)
12. [Running disciplines](#12-running-disciplines)
13. [When you are stuck](#13-when-you-are-stuck)
14. [Quick reference card](#14-quick-reference-card)
15. [Appendix: lineage](#appendix-lineage)

---

## 1. Why this method exists

Most failed investigations fail in the first ten minutes. The engineer hears a symptom, pattern-matches it to something they have seen before, and starts testing a fix. When the fix does not work, they are lost because they never established what the problem actually was, what "working" looks like, or what changed. Everything after that is guesswork dressed up as a diagnosis.

This method front-loads definition. It forces the problem to be written down in a single sentence that the customer agrees with, forces the differences that matter to be identified, and only then allows causes to be proposed, pruned, and proven. The discipline that runs through it is falsification: you do not look for evidence that your favourite theory is right, you look for the test that would prove it wrong.

It works on a critical call where you stay with the customer until service is restored, on a ticket worked back and forth over days, alone at your desk, or at home when the car will not start. The phases are the same; only the pace changes.

## 2. The method at a glance

| # | Phase | Question it answers | Output |
|---|---|---|---|
| 1 | Triage | How do we run this incident? | Severity, mode, mitigation decision, comms cadence |
| 2 | Define | What exactly is the problem? | One-sentence definition the customer confirms, plus "what good looks like" |
| 3 | Scope the differences | What is affected, and what is not? | An IS / IS NOT table and a change timeline |
| 4 | Hypothesise wide | What could cause this? | A long, unjudged list of candidate causes |
| 5 | Surface assumptions | What are we taking on faith? | Assumption list, converted to facts where cheap |
| 6 | Prune | Which candidates survive cheap tests? | A short list of live hypotheses |
| 7 | Prove, fix, verify | Is this really the cause, and is it fixed? | A cause that survived a disproof attempt; a verified fix |
| 8 | Record | What do we keep from this? | RCA with learnings and follow-ups |

*Running disciplines apply across every phase: take notes (AI and manual, reconciled afterwards), screenshot errors and configuration, record every command and its output, request logs and metrics early, change one thing at a time, keep an audit trail, and communicate on an agreed cadence.*

## 3. Two operating modes

### Live call mode

The customer is on the line and stays there until the problem is fixed or worked around. Phases compress into minutes, and Phase 1 dominates: the mitigation decision has to be made early and out loud. Say what you are doing as you do it, agree the comms cadence for anyone not on the call, and capture evidence continuously because you will not get a second chance at the pre-restart state.

### Ticket mode

The same phases stretch over days. Every reply should do two things: close ambiguity from the last exchange and advance exactly one phase. Batch your questions so the customer answers once rather than five times, state which hypothesis a requested test targets so they understand why they are running it, and keep the current one-sentence definition at the top of your internal notes so anyone picking up the ticket starts from the same place.

## 4. Phase 1: Triage

**Goal:** decide how to run the incident before you try to diagnose it.

- Establish severity and impact in numbers, not adjectives. "Production is down" and "one dashboard tile renders slowly" both arrive as "urgent".
- Make the mitigation decision: can service be restored right now by rollback, failover, or restart? If yes, capture evidence first, then mitigate, then continue diagnosis calmly. Restarting a node destroys the state you need to determine the root cause.
- Establish the customer's appetite for change. Some want to hand the problem over and get on with their day; others want to test fixes in production immediately. This determines what you can ask them to run.
- Agree on the communication cadence: who gets updated, how often, and in what channel.

> **Example:** *A cluster is refusing connections. The customer can fail over to the standby region in five minutes. Before they do: grab the tserver logs, the connection counts, and a stack sample from the affected nodes. Then fail over. Root cause work continues without a burning production system in the background.*

> **Pitfall:** Diagnosing while the house burns. If a safe mitigation exists, evidence capture plus mitigation beats heroic live debugging almost every time.

## 5. Phase 2: Define the problem

**Goal:** a one-sentence problem definition that the customer confirms as accurate, plus an explicit statement of what good looks like.

### Let them talk first

Ask the customer to describe the issue in their own words and do not interrupt. This gets the frustration out, earns you the right to ask detailed questions, and gives you their mental model of the problem, which is itself diagnostic.

### Then kill the ambiguity

Get every loaded word defined, even the ones you think you understand. Miscommunication here poisons the whole investigation.

| They say | You ask | You get |
|---|---|---|
| "It's always happening" | How many times today? Do you have earlier tickets? Can I have the numbers? | Frequency, history, and linked tickets 18234 and 18867 |
| "Big impact on production" | What percentage of requests? What fails for the user? | "About 5% of writes time out; checkout retries succeed but add 8 seconds" |
| "The database is crashing" | Crashing how? Process exit, node unreachable, or queries erroring? | "The tserver process restarts; postmaster logs show a shared memory error" |
| "After the upgrade" | Which component, from what version to what version, on what date? | "Smart driver 42.3.5 to 42.7.3, deployed Tuesday to half the app fleet" |

### Build the definition

From the answers, you should now be able to state:

- The one-sentence problem definition.
- The impact and severity are quantified.
- When it started, and what changed around that time: new code, new install, load increase, new connection method.
- What good looks like. You will verify the fix against this later, so make it measurable.
- Whether it has ever worked, and if so, where, when, and what was different.

### Read it back

Read the full definition back to the customer and get explicit confirmation that it reflects the problem they are seeing. This is not a ceremony. It catches misunderstandings while they are still cheap, and it tells the customer they have been heard.

> **Example definition:** *"YSQL read latency at p99 rose from 12ms to 300ms for the orders service in eu-west-2, starting Tuesday 14:00 UTC, shortly after the driver upgrade on half the app fleet. Writes and other regions are unaffected. Good looks like p99 back under 20ms." Confirmed by the customer in one exchange.*

> **Pitfall:** Jumping to cause. The moment someone says "that sounds like the connection pool", the definition work stops, and the investigation narrows before the facts are in. Hold the line: definition first.

## 6. Phase 3: Scope the differences

**Goal:** find the difference that makes the difference. The cause must explain why the problem appears exactly where it does and nowhere else.

### The IS / IS NOT table

Borrowed from Kepner-Tregoe. For each dimension, record what is affected and what plausibly could be but is not. Every IS / IS NOT pair is a constraint that the true cause has to satisfy.

| Dimension | IS | IS NOT |
|---|---|---|
| What | YSQL reads, p99 latency | Writes; YCQL traffic |
| Where | eu-west-2; app hosts on driver 42.7.3 | us-east-1; hosts still on 42.3.5 |
| When | Since Tue 14:00 UTC; constant, not periodic | Before the driver rollout; no nightly pattern |
| Extent | All orders-service queries; roughly half the fleet | Admin console queries via direct connection |

> **Pitfall:** Only recording the IS column. The IS NOT column does most of the work: "unaffected hosts still on the old driver" points at the cause more sharply than anything on the IS side.

### The change timeline

Walk the customer through every change since it last worked, one by one: deployments, config edits, scaling events, certificate rotations, OS patches, load changes. Write them down in order with times. People forget changes they consider unrelated; the timeline surfaces them.

## 7. Phase 4: Hypothesise wide

**Goal:** a long list of candidate causes, obvious and obscure, with no judging.

List everything. Quantity unlocks quality: the tenth idea is often the one that survives, and a visibly long list stops the team from anchoring on the first plausible story. Two techniques help generate candidates:

- **Walk the layers.** Client code, driver, connection pool, load balancer, network, node, storage, and, for a distributed database, the consensus and replication layers too. Ask at each layer: what here could produce this symptom?
- **Interrogate each difference.** For every IS / IS NOT pair, ask what could cause the problem on exactly that side of the line. "What is different about driver 42.7.3?" generates hypotheses that random brainstorming misses.

> **Pitfall:** Filtering while generating. "That can't be it" is a Phase 6 statement. In Phase 4, it goes on the list.

## 8. Phase 5: Surface the assumptions

**Goal:** nothing load-bearing stays invisible.

Every hypothesis rests on assumptions: about versions, topology, configuration, and what the customer actually did. Assumptions are fine; hidden assumptions are how investigations lose days. Write each one down, convert the cheap ones into facts, and flag the rest clearly so everyone knows they are unverified.

| Assumption | Convert or flag | Result |
|---|---|---|
| Cluster is on 2025.1.3.2 | Ask for the version output | Actually, 2025.1.2.1: two relevant fixes are missing |
| The load balancer routes evenly | Check per-node connection counts | One node holds 60% of connections |
| "Nothing else changed" | Pull the deploy and config audit logs | A gflag was changed on Monday, forgotten |
| Retries are idempotent | Cannot verify cheaply | Flagged as unverified, revisit if hypotheses die |

## 9. Phase 6: Prune the list

**Goal:** reduce the long list to a small set of live hypotheses using cheap tests, before anyone commits hours to deep investigation. This is not final confirmation; it is a triage of ideas.

- **Cheapest test first.** A grep of the logs, a version check, or a single metric query can kill half the list in ten minutes. Order tests by cost, not by how much you like the hypothesis.
- **Test against the differences.** A hypothesis that cannot explain the IS NOT column is dead. If "network congestion" cannot explain why direct connections are fine, strike it.
- **Bisect the space.** Halve whatever you can: roll one host back to the old driver, disable one feature flag, replay half the workload. Each split rules out half the remaining candidates. This is `git bisect` thinking applied to production.

> **Pitfall:** Falling in love with a hypothesis. The evidence decides which candidates survive, not seniority or how satisfying the story is.

## 10. Phase 7: Prove the cause, fix, verify

### Prove it critically

For the leading hypothesis, ask the falsification question: if I wanted to disprove this, what would I test? Then run that test. Confirmation is easy to find for almost any theory; a theory that survives a genuine attempt to kill it has earned belief. Then ask: does the cause still look valid based on all facts gathered so far, including the entire IS / IS NOT table?

The strongest proof is reproduction: make the problem happen on demand, then make it stop by removing the suspected cause. If you can toggle the symptom, you have the cause.

### Fix it, one change at a time

Apply the fix or workaround as a single change. If two things are changed together and the symptom disappears, you do not know which one worked, and neither will the next engineer who reads the ticket.

### Verify against "good"

Test the outcome against the definition of good from Phase 2. "P99 back under 20ms" is verifiable; "seems better" is not. Have the customer confirm on their end as well.

### If it is disproven

A dead hypothesis is progress, not failure: the search space just shrank. Return to the list, re-check the assumptions from Phase 5, and go collect more differences. The loop is the method working, not the method failing.

## 11. Phase 8: Record the RCA

The RCA is not paperwork for the customer. It exists for three reasons:

- It creates improvements: monitoring, alerts, docs, or product fixes that prevent the issue or make it faster to diagnose next time.
- It cements what you learned. Knowledge that is not written down evaporates within a month.
- It gives others, the CSM, the account exec, the engineering manager, a fast and accurate picture of what happened without a meeting.

### RCA skeleton

- Problem statement: the one-sentence definition from Phase 2, final version.
- Impact: who, what, how much, for how long.
- Timeline: detection, key findings, mitigation, resolution.
- Symptoms and evidence: what was observed, with the artefacts.
- What looked related but was not: the disproven hypotheses, so the next person does not re-walk them.
- Root cause: the proven cause and the proof.
- Fix and workaround: what was done, and the verification against "good".
- Learnings and follow-ups: each with an owner.

## 12. Running disciplines

- Take notes throughout, by AI and by hand where possible; reconcile the two after the call.
- Capture screenshots of errors, commands, and configuration as you go; clean them up afterwards, not in the moment.
- Record every command run during a session and its output. This is your audit trail and the raw material of the RCA.
- Request logs, metrics, and artefacts early, before restarts or failovers destroy them, and before an escalation needs them.
- Change one thing at a time, always.
- Communicate on the agreed cadence even when there is no progress to report. Silence reads as absence.

## 13. When you are stuck

Grinding alone for hours is not diligence; it is ego. Set a timebox when you start a line of investigation; when it expires with no progress, stop and do one of three things: bring in a second pair of eyes, escalate, or step back to Phase 2 and re-verify the definition, because long-stuck investigations are very often stuck on a wrong or incomplete definition.

When you escalate, hand over the package this method has already built: the confirmed problem definition, the IS / IS NOT table, the change timeline, the assumption list, and the hypotheses tested with their results. An escalation with that package gets picked up and moves within minutes. An escalation that says "customer says the database is slow" goes to the back of someone's queue.

## 14. Quick reference card

| Phase | Do | Don't |
|---|---|---|
| 1 Triage | Quantify severity; decide mitigate-now vs diagnose; capture evidence before any restart; agree comms cadence | Debug live while a safe mitigation waits |
| 2 Define | Let them talk; define every loaded word; one sentence; "what does good look like"; read it back | Jump to cause; accept "always" or "big impact" unquantified |
| 3 Differences | Build IS / IS NOT; walk every change since it last worked, with times | Ignore the IS NOT column |
| 4 Hypothesise | List everything; walk the layers; interrogate each difference | Filter while generating |
| 5 Assumptions | Write them all down; convert cheap ones to facts; flag the rest | Let anything load-bearing stay invisible |
| 6 Prune | Cheapest test first; kill anything that cannot explain the IS NOT facts; bisect | Deep-dive a favourite before cheap tests run |
| 7 Prove and fix | Ask "what would disprove this?"; reproduce if possible; fix one change at a time; verify against "good" | Declare victory on "seems better" |
| 8 Record | RCA with disproven paths, learnings, and owned follow-ups | Treat the RCA as paperwork |

## Appendix: lineage

[Kepner-Tregoe](https://kepner-tregoe.com/training/problem-solving-decision-making/) is the one framework in this list I actually trained on; it is where the IS / IS NOT table comes from. The rest are not claimed sources, just recognisable overlap for anyone who already knows them:

- **Kepner-Tregoe Problem Analysis:** the IS / IS NOT table and a focus on distinctions and changes.
- **The scientific method:** falsification as the standard of proof, per Popper. You believe in a cause when it survives an attempt to kill it.
- **Ford 8D:** containment before correction, and institutionalising the learning afterwards.
- **Differential diagnosis:** list broadly, rule out cheaply, confirm decisively, from medicine.
- **David Agans, *Debugging: The 9 Indispensable Rules*:** quit thinking and look; change one thing at a time; keep an audit trail.
- **CompTIA's six-step troubleshooting method:** the version many support engineers already know; this method deepens it rather than replacing it.

---

*Feedback, corrections, and pull requests welcome. If you use this and it saves you time on a call, that's the whole point.*

## License

This work is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Use it, adapt it, teach it, just credit the source.
