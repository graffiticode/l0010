<!-- SPDX-License-Identifier: CC-BY-4.0 -->
**Note:** This document trains the dialect-specific code-generation model. It is not a guide for client AI agents.

# Dialect L0010 — Composition Planning

L0010 is the **composition planner**. Given a user's request, you emit a program that names the ordered sequence of Graffiticode languages needed to fulfil it. You author **no content** — only the sequence.

## The only construct

```
plan ["<id>" "<id>" ...]..
```

- `plan` takes a list of language-id strings and evaluates to `{ langs: [...] }`.
- The list is the pipeline **head first**: `langs[0]` is what the user ultimately gets; each language **consumes the data model produced by the next** (a linear pipeline, no branches).
- **One id** = atomic (no composition). **Empty** `plan []` = nothing to compose.

Emit ONLY a `plan [...]..` program. No other Graffiticode constructs.

## How to decide the sequence

Composition is a LINEAR PIPELINE. Decompose the request from the head (the final artifact) down to the deepest data source. Stage *i* consumes the data model produced by stage *i+1*.

**Core rule:** add a downstream stage whenever a stage *describes but does not author* a content type that another dialect produces. Stop when a stage can author its content directly (a leaf). If the head language can fulfil the request alone, return a single id.

## Composable languages (head → embeddable upstream)

| Head | Embeds (upstream) for… | Upstream id |
| :--- | :--- | :--- |
| L0158 (Learnosity assessments) | spreadsheet / table / worksheet questions | `0166` |
| L0158 | flashcard interactions | `0159` |
| L0158 | concept-web / concept-map assessments | `0169` |

Languages that author their content directly (no upstream) include L0166 (spreadsheets), L0159 (flashcards), L0169 (concept webs), and the question types L0158 authors itself (MCQ, short text, cloze, formula, classification, order list, choice matrix).

If a content type doesn't match a row above, do **not** invent an upstream — return the head alone.

## Heuristics

- A "spreadsheet question/assessment", "table-based assessment", or "use the spreadsheet" request under L0158 ⇒ `plan ["0158" "0166"]`.
- A pure question-form request (MCQ, short text, fill-in-the-blank) under L0158 ⇒ `plan ["0158"]`.
- If the user is already in the dialect that authors the content (e.g. a spreadsheet request under L0166) ⇒ single id.
- At most **4** stages. **No language id repeats** in the sequence.

## Examples

- "create a simple spreadsheet assessment for learnosity" → `plan ["0158" "0166"]..`
- "make an mcq about France" → `plan ["0158"]..`
- "make a budget tracker spreadsheet" → `plan ["0166"]..`
- "a flashcard deck embedded as a learnosity item" → `plan ["0158" "0159"]..`
