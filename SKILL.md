---
name: hana-ask-user
default-enabled: true
description: "Cast a question card when the Agent must pause on the user's decision: confirm something, choose between options, or supply missing information, in one short set. Use when the next step depends on the user's answer, or when several short questions belong together."
profile: card-skill
---

# Ask user question

A Hana-card replica of the DeepSeek Harness `ask_user_question` tool. The
Agent ends its turn by casting a question card; the user answers the set;
the card emits the answers back into the conversation and wakes the Agent.

Question text, option labels, and free-text answers are the conversation's
words, not this document's: write every user-facing string in the user's own
language, whatever language the examples below happen to use.

## When to cast

- The next step depends on the user's decision: confirm, choose, or supply a
  missing fact the Agent cannot look up itself.
- One to four short, self-contained questions share one card.

Not for: long surveys or forms with many fields (write a custom card),
anonymous group voting, or anything the Agent can answer by reading files or
searching instead of asking.

## Casting ends the turn

`show_card` returns as soon as the card is in the stream; it does not block.
Casting an ask is therefore a **turn-ending action**: make the card the last
thing you do, write one line saying the card is out and waiting for an answer,
and stop.

In that same turn, do not:

- answer the questions yourself, or settle for a "likely" answer so the work
  can continue;
- assume any option is selected, or start anything that depends on the answer;
- cast a second ask while one is still unanswered;
- poll, re-cast, or re-ask to check whether the answer has arrived.

The answers come back as the `hana-ask-user.answer` card event, which
wakes you in a **new** turn. Until that event arrives the ask is unanswered:
resume from the event's payload, never from a guess.

## Template

`hana-ask-user/assets/ask.card.html`

The mint's `state` replaces the template defaults; send the whole object:

```
{
  "uiLanguage": "en",            // from the conversation: zh, zh-TW, en, ja, ko
  "callId": "dinner-plan",       // stable id, unique among this session's asks
  "header": "Tonight",           // optional card heading
  "asker": "Hanako",             // optional: the asking Agent's own name, in the card's language
  "questions": [
    {
      "id": "dinner",            // stable, unique within the set
      "question": "What should we have for dinner tonight?",   // self-contained, user-facing words
      "header": "Dinner",        // optional per-question label
      "options": [               // optional; omit for a free-text answer
        { "label": "Cook at home (Recommended)", "description": "Simple, and we eat sooner" },
        { "label": "Order in", "description": "No dishes, a little more expensive" }
      ],
      "multiSelect": false       // optional
    },
    {
      "id": "note",              // stable, unique within the set
      "question": "Anything you want with it?"    // no options: a free-text answer
    }
  ]
}
```

Mint conventions:

- If you recommend an option, put it first and append the recommendation
  marker of the card's language to its label — "(Recommended)" in English,
  "（推荐）" in Chinese, and the equivalent elsewhere; the card does not infer
  recommendations itself.
- `asker` is optional: the name this Agent goes by in the conversation's
  language. The card writes it into its own title and, when it is omitted,
  falls back to that language's default name (小花 / Hanako / 花子 / 하나코).
- `submitted` belongs to the card, not to the mint: after Submit the card
  writes its own record of the set (`answers`, `at` — a local `YYYY-MM-DD HH:MM`
  stamp) into that state field, so a reopened card shows what was answered. Do
  not send it.
- Questions without `options` render a free-text field; with `options` the
  free text becomes an optional supplement or custom answer.
- Keep question ids stable: they are echoed in the answer and let you match
  an answer back to the ask that produced it.
- Cast one ask at a time; the set is the unit of asking.

## The answer event

On submit the card emits `hana-ask-user.answer` and the Agent wakes:

```
{ "callId": "deploy-check",
  "answers": [ { "id": "regression", "selected": ["Run it first (Recommended)"], "custom": "Also check the bundle size" } ] }
```

- `selected` holds option labels; `custom` holds free text. `selected` is an
  array, empty when nothing was picked; `custom` is omitted when there is no
  text.
- An answer item with empty `selected` and no `custom` means the user
  deliberately skipped that question — a completed set, not silence. Do not
  re-ask.
- The card walks the questions one at a time, as DSH does: each page holds a
  single question, with a pager when the set has more than one. The bottom
  bar is Skip on the left and Previous/Next on the right; on the last page
  Next gives way to Submit.
- The whole set is submitted at once; there is no partial submit. Skip
  (bottom-left) marks the current question empty and advances; tapping it
  again on a marked question resumes it. Skipped questions submit as empty
  answers (empty selected, no custom); the rest keep their answers.

## Outside a host

Standalone browsers render the questions read-only with a note that
delivering answers needs the Hana host; nothing is sent anywhere.
