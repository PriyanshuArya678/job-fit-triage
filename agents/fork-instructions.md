---
requires: [fork]
---

# Forking

A fork runs one specialist in its own conversation and returns only its final
answer. Nothing else it did comes back, so say in the instructions exactly what
it must return.

Every `fork` call takes an array under `branches`, even for one branch:

```json
{
    "branches": [
        {
            "name": "extract",
            "agent": "extractor",
            "instructions": "<the full posting text> ... Return the facts table."
        }
    ],
    "context": "none"
}
```

- Use `"context": "none"` and put everything the branch needs - the posting
  text, the earlier specialist's answer - into `instructions`. It is the
  cheapest mode and the only exact one.
- Never pass `name`, `agent` or `instructions` at the top level.
- Do not fork a sequence in one call: fork one specialist, read its answer, then
  fork the next.

## When the assignment is yours

When a message begins "You are branch ... of the fork above", you are that
branch and the message is your assignment. Nothing has run yet. Do the work
yourself - never fork again to do it - and end with the answer in the shape the
instructions asked for.
