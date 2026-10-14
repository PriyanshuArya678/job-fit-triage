# job-fit-triage

A [ZeneraNeo](https://github.com/andreyryabov/ZeneraNeo) project that decides
whether a job posting is worth applying to - and drafts the application when it
is.

I built it for my own job search. Applying by hand, the same rules kept being
applied inconsistently: a "1-3 years" headline whose body asks for 3+, a
"full-time" role that is really a 3-month contract, a salary below the floor in
the last paragraph. This project makes every one of those checks explicit,
quoted and logged.

## How it works

`SPECIFICATION.md` is the source of intent; `agents.yaml` and the prompts
implement it. Four agents, one model:

```
triage (entry)
  ├─ fork → extractor   posting → facts table, every value with its quote
  ├─ fork → screener    facts + rules → APPLY / SKIP / ASK, rule table, fit score
  └─ fork → writer      (APPLY/ASK only) ≤120-word note + screening answers
  └─ writes reports/<company>-<role>.md and a line in reports/decisions.csv
```

- **Forks, not hand-offs.** Each specialist runs in its own conversation and
  returns only its answer, so the triage agent's context holds three
  conclusions instead of three transcripts.
- **Evidence or `unknown`.** A rule with no quote from the posting is reported
  `unknown`, never a pass; an unknown on a decisive rule (experience,
  employment type) turns the verdict into ASK instead of guessing.
- **Least privilege.** Only `triage` can write, and only reports; the
  specialists are read-only. No shell, no network, no sandbox needed.
- **Rules and candidate are data.** `assets/rules.md` and `assets/profile.md`
  are plain Markdown mounted read-only at `/assets` - change the rules there,
  not in a prompt.

## Run it

```sh
npm i -g @zenera/cli
export OPENAI_API_KEY=<groq key>   # Groq is OpenAI-compatible; or put it in .env (git-ignored)
zen init .                     # registers this cloned folder
zen check .
zen run . "triage examples/apply-ai-platform.md"
zen inspect                    # the run's trajectory: forks, tool calls, tokens
```

`examples/` holds three postings with known answers in `examples/EXPECTED.md`:
one APPLY, one SKIP on a disguised contract, one SKIP on a body that contradicts
nothing but asks for 3+ years.

## Files

| Path | What it is |
| --- | --- |
| `SPECIFICATION.md` | What the system must do - the source of truth |
| `agents.yaml` | Agents, model, tool grants, fork wiring |
| `agents/prompts/*.md` | One role prompt per agent |
| `assets/rules.md` | Fields to extract and the hard rules, in order |
| `assets/profile.md` | The candidate |
| `assets/writing-style.md` | How notes are written |
| `examples/` | Test postings and expected verdicts |
| `reports/` | Output: one report per posting, plus `decisions.csv` |

## Notes

- **Groq free tier.** The project runs on `openai/gpt-oss-120b` via Groq, whose
  free tier caps a request at 8,000 tokens per minute and 200K tokens per day.
  To fit, the stock `agents/fork-instructions.md` and
  `agents/files-instructions.md` are trimmed to the parts these agents use (so
  `zen check` warns that they differ from the stock files), the house rules are
  two lines, and the specialists keep quotes to 15 words and answers under 300
  words. On a paid tier or another provider, change the model in `agents.yaml`
  and none of that is needed.
- **No secrets in the repo.** The key lives in `.env`, which is git-ignored.
