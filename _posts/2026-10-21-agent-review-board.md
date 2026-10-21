---
layout: post
title: "A document is a better place for a branching conversation"
description: "Why I built a local review board for working with coding agents, and what actual use changed."
---

I would ask an agent five questions. It would answer them and introduce three
more. My next message would refer to questions one, three and five, plus two of
the new ones. After a few rounds, even saying which question I meant became work.

The discussion had become a tree. The interface was still a stream.

[Agent Review Board](https://github.com/willabel4112/agent-review-board) grew out of
that problem. It renders a local Markdown document, lets me comment on a passage,
and gives my existing coding agent a queue of delivered questions. The agent can
answer in a thread or edit the source document. I can inspect the change and
decide whether to accept it.

I am publishing the current implementation, a runnable synthetic demo, and the
lessons that shaped it.

![A synthetic review in the actual application](/assets/agent-review-board/review.jpg)

## The first request was about keeping my place

I wanted to take an existing project document, open it in a local webpage, and
comment where I was reading. Large projects had become difficult to hold in my
own working memory. Terminal output was also a poor place to inspect plans,
diagrams and explanations with several levels of detail.

The initial choices were modest: a persistent local service, an agent session in
the loop, and comments rather than a browser document editor. Markdown stayed in
its repository. The agent changed the source; the page reflected the result.

The familiar precedent is a document margin comment or a pull-request review.
The object here is a living working document, with both explanatory questions and
requests to change it. Those two kinds of work need different endings.

## A report became useful through its questions

One early success was a substantial research report. It accumulated dozens of
anchored questions and several waves of follow-up. Some answers were short. Others
became linked documents. A follow-up could refer to another answer without forcing
the whole discussion back into one chat message.

Several first-draft judgments changed during that review. One conclusion moved
from a discussion thread into a maintained policy document. I explicitly called
that session a success at the time: I could not see how I would have managed the
same questioning and refinement comfortably in the old interface.

This is an account of use, not a controlled productivity result. The useful
observation is narrower: the first draft was the start of the work, and a stable
place for each question made the subsequent work manageable.

## Successful feedback broke the original anchors

Another review exposed a less flattering result. A plan changed substantially,
but the board showed little more than short acknowledgments. I could not easily
see what my comments had changed.

Some comments also lost their anchors because the passages they referred to had
been rewritten. The feedback had succeeded, and the UI presented that success as
a broken reference.

That led to source revisions, content-addressed snapshots, forward mapping of old
blocks, and block-level diffs. A review round could then have a meaningful ending:
the human accepts the resulting version, or sends it back.

There is a second failure that a better diff cannot solve. While reviewing the
board's own specification, I wanted to work together on the document. Too many
agent responses only explained things in comment threads. If the source never
changes, there is nothing useful for the diff to show.

The agent instructions now require an accepted specification change to reach the
canonical document. The thread keeps the explanation and a reference to the edit.
An answer and an applied change are different outcomes.

## Three boundaries that matter

**A draft is not a message.** I can send one question immediately or collect a
batch. Unsent comments and private notes stay out of the normal agent queue.
This is a workflow convention, not a security sandbox: the underlying files
remain on my machine.

**Silence is not acceptance.** Commenting on one paragraph says little about the
rest. The board has explicit read and approval actions. A rewritten block becomes
unread again. Scrolling, opening a branch and receiving an answer do not approve it.

**Navigation is not model memory.** The latest version adds a question tree and a
reading stack. I can explore a child document and return to the place I left,
including unsent input while the page remains open. The tree does not fork an
agent, assemble its prompt or guarantee context isolation.

![A child document opened from its parent question](/assets/agent-review-board/explore.jpg)

## What is in the release

The first public package is based on the working v2.7 feature set. It includes
selection and block comments, follow-ups, batch review, version comparisons,
search, reading coverage, local session claims, and the exploration view.

The server and CLI use Python's standard library. Browser dependencies are
vendored. There is no model API, API key or automatic answering service: the board
connects to the agent workflow you already run. The interface is currently Chinese;
the English guide includes a button glossary.

```sh
git clone https://github.com/willabel4112/agent-review-board.git
cd agent-review-board
python3 examples/demo.py
```

Open the printed URL. The demo creates a fictional search-tool evaluation plan,
two review threads, a changed paragraph and a child document in a temporary
directory. It does not load an existing registry or any personal documents.

The public tests exercise rewritten anchors, review rounds, draft exclusion,
exploration relationships, navigation helpers, portable session identity and the
HTTP/CLI reply loop. The screenshots here show that synthetic demo. The newest
exploration view has less accumulated usage evidence than the older review loop.

The release starts a clean Git history. Original documents, comments, private
notes, machine inventories, session history and operational test data are excluded.
Historical cases are paraphrased, identifying detail is omitted, and exact usage
counts are generalized. This is a local single-user tool, not a public web service.

The [code and guide](https://github.com/willabel4112/agent-review-board) are available
under MIT, with upstream dependency licenses preserved. The
[case notes](https://github.com/willabel4112/agent-review-board/blob/main/docs/ORIGIN-AND-CASES.md)
separate historical experience from mechanisms that can be checked in the package.

The question I want this tool to answer is simple: after a good discussion with
an agent, can I find what we decided, see what changed, and resume the right branch?
