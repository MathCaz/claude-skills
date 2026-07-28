---
name: grill-me-M
description: Interview the user relentlessly about a plan, design, or topic until reaching shared understanding. Walks each branch of the decision tree. Decisions are captured in a distilled brainstorm file (delegated to notes-M). Use when user wants to stress-test a plan, get grilled, run a discovery session, or says "grill me".
disable-model-invocation: true
---

# Grill Me

Relentlessly interview me about every aspect of the topic until we reach shared understanding. Walk down each branch of the decision tree, resolving dependencies one by one. Goal: extract what's in my head into a durable, organized file.

## File mechanics — delegated to `notes-M`

All file operations (create / reopen / structure / checkpoint / curation / status / reconcile) live in the **`notes-M`** skill. Use it in **`interview` mode**.

`notes-M` will auto-invoke when it detects a multi-step interview. You don't need to call it explicitly — just follow the interview discipline below, and write each captured decision to the file `notes-M` creates.

Body section in the file = `## Decisions log` (see `notes-M` for the consolidated-entry format: Decision · Why · Dead-ends · Flags).

## Question style

For each question:
- Present **numbered or lettered options** whenever possible so I can reply with just a number, letter, or one word.
- For open-ended questions that can't be simplified, mark them clearly as needing more detail.
- Provide your **recommended answer** (your best inference from context) so I can just confirm, correct, or redirect.
- Minimize my typing — aim for replies of one word, one number, or 1–2 sentences max.

## Question flow

- Ask **one question at a time**. Wait for my answer before moving on.
- Resolve dependencies in order: settle the upstream decision before ones that depend on it.
- If a question can be answered by **reading a file/doc or exploring the codebase**, do that instead of asking. If I hand you a doc (Google Doc, etc.), read it and surface only what's net-new.
- When I **can't answer** something, capture it as a flag (item → owner) in the notes file and move on. Don't stall.
- Keep going until I say we're done or you've covered every branch. Offer a completeness backstop near the end ("anything we haven't touched?").

## Checkpoint after every answer

Non-negotiable: after EVERY answer, BEFORE the next question, update the notes-M file with the new decision/flag. Apply `notes-M`'s curation rules (collapse, supersession, pivot). Never batch.

## At the end

When the interview reaches a conclusion (or is being parked), trigger `notes-M`'s end-of-topic reconcile: read the file for contradictions, fix them, set status to `concluded` or `parked`, give me a short recap.
