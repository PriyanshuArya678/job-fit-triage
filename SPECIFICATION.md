# Specification

What this project is for. The files around it are the implementation; where the
two disagree, this one wins.

## Purpose

Decide whether one job posting is worth applying to for one specific candidate,
and if it is, prepare what the application needs.

Given a job description (a file in the workspace, or text pasted into the
request), the system returns a verdict - **APPLY**, **SKIP** or **ASK** - with
every reason backed by a quote from the posting, plus, for APPLY or ASK, a short
tailored note and answers to the screening questions the posting implies.

It exists because a job search is recurring work with hard rules that are easy
to apply inconsistently by hand: a "1-3 years" headline that the body
contradicts, a "full-time" role that turns out to be a contract, a salary below
the floor buried in the last paragraph.

## Scope

- One posting per request. A request naming several files is triaged one file
  at a time, each with its own report.
- The candidate is the one described in `assets/profile.md`; the rules are the
  ones in `assets/rules.md`. Neither is inferred from the posting.
- No applying, no messaging, no browsing: the system reads files and writes
  reports in the workspace, and nothing else.

## Models

- `claude` - `anthropic:claude-sonnet-5-5`, one model for the whole project,
  used by every agent. Named once in `agents.yaml` so it can be swapped in one
  place; the prompts depend on no particular model.

## Setup

- Currently: no steps. Nothing has to be prepared before a run.

## Agents

- `triage` - the entry point. Reads the posting, forks the three specialists
  below in order, and writes the report. It never decides a rule itself; it
  assembles what the specialists return.
    - Forks `extractor` first, with the full posting text.
    - Forks `screener` with the extractor's facts.
    - Forks `writer` only when the screener's verdict is APPLY or ASK.
    - Writes `reports/<company>-<role>.md` (lower-case, hyphenated) and appends
      one line to `reports/decisions.csv`.
- `extractor` - turns a posting into facts. Returns, for each field in
  `assets/rules.md` § Fields, the value and the exact sentence it came from, or
  `not stated`. Never guesses a field the posting does not state.
    - When the headline experience range and the body disagree, it reports both,
      quoting each.
- `screener` - applies `assets/rules.md` to the extracted facts and
  `assets/profile.md`.
    - Every hard rule is reported as pass, fail or unknown, with the quote that
      decided it.
    - Any fail -> SKIP. No fail, but an unknown that a hard rule depends on ->
      ASK, naming what to find out. Otherwise -> APPLY.
    - Adds a fit score 0-10 from skill overlap with the profile, listing the
      matched and the missing skills.
- `writer` - drafts what an application needs, following
  `assets/writing-style.md`.
    - A note of at most 120 words, specific to this company and role.
    - Answers to the screening questions the posting implies (notice period,
      relocation, years with a named technology), from `assets/profile.md` only.
    - Anything the profile does not support is left as a question for the
      candidate, never invented.

## Tools and boundaries

- Files - read and search for every agent; create and edit only for `triage`,
  and only under `reports/`.
- `assets/` is reference material, and read-only.
- No shell and no network: everything needed is in the posting and the assets.

## Environment

- No sandbox and no memory store. `reports/decisions.csv` is the project's
  record across runs.

## Testing

- `zen check` passes.
- `examples/` holds three postings with a known answer, listed in
  `examples/EXPECTED.md`. A run on each produces the expected verdict and names
  the rule that decided it.

## Done means

- `reports/<company>-<role>.md` exists and contains: the verdict, the rule table
  (rule, result, quote), the fit score with matched and missing skills, and for
  APPLY or ASK the note and the screening answers.
- `reports/decisions.csv` has one new line for the posting.
- Every reason in the report quotes the posting; a rule with no quote is
  `unknown`, not a pass.
- The reply names the report path and the verdict in its first line.
