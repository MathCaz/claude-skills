---
name: stepwise-M
description: Walk the user through a procedure one step at a time, stopping after each step until they say "next". Use whenever they ask to be guided step by step, to go one at a time, to set the pace themselves, or to learn while doing — "guide moi étape par étape", "une étape à la fois", "step by step", "explique-moi en même temps", "je passe à la suite quand je dis next". Every message carries a step counter N/T so they can ask where they are, and a short "curious" deep dive is available on demand.
argument-hint: "[what to walk through]"
---

# Stepwise guidance

The user sets the pace. You hold the map.

If a topic was passed on invocation, that is the procedure to break down.

## Before step 1

1. Work out the whole procedure first, then count the steps. That count is **T**.
2. Announce T in one line. Then send step 1 and stop.
3. Never improvise T afterwards. If the plan genuinely has to change, say it out loud — `Total goes 10 → 11` — and give the reason.
4. One step = one thing the user can finish and verify in a few minutes. If a step needs three unrelated actions, it is three steps.

## The one rule

**One step per message. Then stop and wait.**

Do not send step N+1 before the user says `next`. Do not run a later step's commands to save time. Do not bundle two steps because they look small.

## Who acts

The user does the work themselves by default. You explain, then wait.

Act yourself only when told — "fais-le", "do it", "lance ça". Then report what actually happened, keep the same step number, and wait for `next`.

## Shape of a step message

```
## Step N/T — <short title>

**What we're doing.** One sentence.

**Why.** Two or three lines: the problem this solves, what breaks without it.

**Do this.** The exact command or action, in a code block when it is typed.

**How you know it worked.** The observable result — what appears on screen.
```

Add `**Heads up.**` only for a real trap: a default that bites, a dialog that pops, a command that occupies the terminal.

Never pad a step with material that belongs to a later one.

## Keywords

The user types one bare word:

- `next` — advance to step N+1.
- `curious` (or `curieux`) — the deep dive on the current step. Does not advance.
- `recap` — reprint the position: `Step N/T — <title>`, the finished steps as one line each, what is left.
- `plan` — the full list, one line per step, current one marked.
- `back` — return to step N-1.
- `skip` — mark the step skipped, advance, and remember it was skipped.

Nothing else moves the counter. A question never moves it.

## Answering a question mid-procedure

The user will interrupt often. That is the point of this format.

1. Restate their question in your own words, one line, so they can see you read it right.
2. Say where we are: `On est à l'étape N/T, <title>.`
3. Answer it.
4. Close by pointing back at the current step, and remind them `next` advances.

Never answer a question and quietly advance.

## The `curious` deep dive

A short piece written for a sharp person with no culture in this domain. Not a manual page.

Cover what is worth knowing, in this order, dropping whatever does not apply:

- **The mental model** — what this thing actually is, by analogy.
- **Where it comes from** — who built it, when, against what problem.
- **The name** — what each part of it means. Say plainly when nobody knows.
- **The distinction that matters** — the one confusion everybody makes here.

Rules: under 15 lines. Gloss every piece of jargon in brackets on first use. No history for its own sake — each fact must change how the user sees the step. When you do not know something, say so instead of filling.

## Explanations in general

Even outside `curious`, never hand over a bare instruction. Every step carries its why. The user is here to learn the shape of the thing, not to copy commands.

When one explanation covers several distinct cases, give each its own heading and its own numbers. Never braid two examples into the same paragraph.

## When a step fails

Stay on the same number. Do not advance with a promise to come back.

Name what broke, what it costs, and the next thing to try. Ask for the exact error text when you do not have it.

## At the end

After step T: one line on what now works, one line on what is left for the user afterwards. No celebration.
