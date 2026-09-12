---
layout: post
title: "Interview your handoff document before you leave"
---

Every agent session ends the same way for me: I write a handoff so the next one can continue.
Goal, what is closed, what is open, traps, the first thing to do. Then the next session starts,
reads it, and gets stuck on the first action — which machine? which of the two configs? "the usual
key", which one?

The document is not bad. The writer just cannot see which of their assumptions are obvious only to
them. That is the curse of knowledge, and no amount of "be comprehensive" in the prompt fixes it,
because the writer is the one judging what is comprehensive.

So we stopped asking the writer. We ask a reader who knows nothing.

## The probe

[`handoff-probe`](https://github.com/willabel4112/handoff-probe) spawns a small, cold agent that
sees only two things: the goal of the next session and the handoff text. No tools, no repository,
no memory of the session that wrote it. It has to do two things: write the first-action plan it would
execute, and list only the questions that would block that plan.

Then the writer — who is still around, with the whole context still warm — answers. Not from memory:
from the repo, the git log, the notes. The answers go into the document, not into the chat. Run it
again. Every round is a fresh probe with no memory of the last one, so convergence means exactly
what it should: a fresh reader finds nothing blocking. It prints `NO-BLOCKING-QUESTIONS`; two of
those in a row and you are done.

A round is one small model call, typically under a dollar and about two minutes. A well-written
handoff is dry in two rounds. A rushed one takes four and yields two or three real holes.

## Two rules that keep it honest

The script is a hundred lines. The rules around it are what make it work.

**Answer from the repo, never from your head.** If the writer answers from session memory, the
document does not improve; the chat does, and the chat dies.

**A question built on a false premise counts as a dry round.** The probe is rewarded for finding
questions. Follow it and it will manufacture blockers out of thin air. "False premise" is a valid
answer, and it means the document was fine on that point.

There is a third one we learned the hard way: read the probe's own summary, not just its questions.
In one knowledge-transfer run the probe declared two rounds dry, but its summary had misread a
measured comparison value as a pass threshold and drawn the opposite conclusion. It never phrased
that as a question. It was still a real bug in the document.

## Where it sits

This is the cheap layer. Predictable gaps die here, while the writer is alive and the context is
warm. What survives are the questions that only surface when the successor actually starts
working — for those you still need to go back and ask the old session (forking it works), and that
is expensive enough that you want to batch the questions. The probe's job is to make that rare.

The ideas are not new. Rawls' veil of ignorance: only a reader who does not know what the writer
knows can judge the document fairly. Hallway usability testing: grab someone who has never seen the
thing. What is packaged here is just the mechanics — an isolated, stateless, read-only reader, a stop
criterion, and the two rules.

## What we do not know

We have run this a hundred-odd times over two months across a few machines. It stayed because
handoffs stopped failing on the first action. There is no A/B test and no benchmark behind that
sentence, and we would like there to be. If you try it, and especially if it does not work for your
kind of handoff, tell us — the repo has Discussions open. Better stop criteria, counter-examples and
comparisons with other approaches are exactly what we are after.

The default backend is Claude Code; any agent CLI that takes a prompt and prints text can be
plugged in with one environment variable, and later releases will treat other harnesses as first
class.

---

*About this account: it is run jointly by a human operator and the coding agents they work with.
The agents draft what gets published from the operator's notes and decisions; the human decides what
ships and answers for it. Pseudonymous for now for ordinary reasons, not because anything here is
secret.*
