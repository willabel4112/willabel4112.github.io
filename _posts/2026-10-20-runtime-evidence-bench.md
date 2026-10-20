---
layout: post
title: "Turning a recording failure into an agent test"
description: "A small, hardware-free case for testing evidence, lifecycle repairs, and recovery policies."
---

A recording app stopped while it was supposed to keep recording. The investigation
produced several plausible explanations, a few useful fixes, and a recovery path
that could make a temporary problem permanent.

I have turned that experience into the first case in
[Runtime Evidence Bench](https://github.com/willabel4112/runtime-evidence-bench):
a small development suite for testing how coding agents diagnose and repair
runtime failures.

The public evidence is a **synthetic reconstruction**. Names, timings, counters
and identifying history have been changed or omitted. There are no recordings,
private conversations, original system logs or application binaries in the
release. The failure mechanisms are preserved in executable tasks.

## A recovery path that removes its own precondition

The most useful failure pattern is easy to describe.

An input stream feeds a finite buffer pool. A serial writer consumes those
buffers and encodes audio. If the writer stalls, the pool eventually fills.
The application treats exhaustion as a capture failure and tries to recover
by stopping and restarting the input engine.

That stop can remove the recording runtime assertion. The subsequent attempt
to start recording from the background is then denied.

```text
writer stalls
    → buffer pool fills
    → recovery stops the live input engine
    → recording runtime is lost
    → background restart is denied
```

The local recovery policy has destroyed a condition it needs to recover.

This also exposes an accounting mistake: if the input heartbeat advances only
when the writer finishes, a writer stall looks like a dead microphone. Those
are different observations and may need different responses.

The repair contract is deliberately bounded. Keep the input path alive through
temporary writer backpressure, bound memory, account for dropped samples, and
preserve the real gaps in the output timeline. A finite buffer cannot promise
lossless capture through an arbitrarily long writer stall.

## Three tasks, one case family

The suite separates the work into three tasks.

| Task | What the agent must do |
|---|---|
| Evidence diagnosis | Classify supported, contradicted and unresolved claims; cite evidence; explain the causal chain and propose a discriminating test. |
| Lifecycle repair | Handle a resource that may become invalid between uses, report failed starts truthfully, and avoid target-width timestamp overflow. |
| PCM writer repair | Survive bounded-buffer backpressure, separate input and writer progress, and recover without hiding gaps. |

The first two run with Python. The third uses Swift and AVFoundation on macOS,
feeding generated PCM into a real audio writer. It needs no microphone permission,
watch, phone, signing certificate or network.

The diagnosis task includes several tempting shortcuts: treating an IO thread
as proof of a playback strategy, using an earlier snapshot to explain a later
failure, or treating a drained delivery queue as proof of continuous capture.
Each asks the agent to keep its conclusion within the evidence.

## What has actually been checked

The suite has 36 passing self-checks. The lifecycle reference passes its 23
checks, while the defective starter is rejected.

The Swift reference passes tests that encode an hour of generated audio into
chunks and verify decoded sample counts, timestamps and resampling continuity.
Additional tests deliberately block the writer twice, check bounded memory and
input progress, and verify explicit gaps after recovery.

The negative control passes ordinary writer checks and fails the backpressure
test. That distinction matters: an uncomplicated recording test can remain green
while the recovery policy is broken.

These are checks of the suite and its controls. **No new model comparison has
been run for this release.** The tests do not establish whole-device battery
life, real background scheduling or watch-face behavior.

## Comparing a model with a system

A historical before-and-after comparison is hard to interpret when the later
agent received more evidence, clearer requirements and a list of failed attempts.
That is useful engineering progress, but it does not isolate a model effect.

The [protocol](https://github.com/willabel4112/runtime-evidence-bench/blob/main/PROTOCOL.md)
therefore distinguishes:

- Changing the model while holding tools, evidence, permissions and budget fixed.
- Comparing complete native agent systems under the same task and acceptance rules.
- Changing one interface or feedback feature while holding the model fixed.

Unknown model identities stay unknown. Failed attempts and human interventions
stay in the record. Machine checks cover only part of acceptance; explanations
and code still need independent review.

This follows a broader line of work:
[SWE-bench](https://arxiv.org/abs/2310.06770) evaluates real repository repairs,
and [SWE-agent](https://github.com/SWE-agent/SWE-agent/blob/main/docs/background/aci.md)
examines how the agent-computer interface affects performance. The contribution
here is a small, inspectable case about runtime evidence and recovery invariants.

## Try it

Start with one exported task:

```sh
python3 bench.py prepare lifecycle trials/lifecycle-01
```

Give the agent only that directory. Keep the evaluator and reference solutions
in a separate environment, then grade the returned work.

The code, fixtures, reference solutions and rubric are
[available under MIT](https://github.com/willabel4112/runtime-evidence-bench).
Because the answers are public, this is a development and regression case,
not an uncontaminated held-out benchmark.

I would like to collect more cases where the hard part is choosing what to
believe and how to verify it. Counterexamples to the labels, repairs that pass
the tests while violating the intended contract, and carefully controlled runs
would all help improve this one.

*Drafted with coding-agent assistance from a reviewed case.*
