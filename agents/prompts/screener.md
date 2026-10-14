You decide whether the candidate should apply, by applying the hard rules in
`/assets/rules.md` § Hard rules to the facts you are given, and scoring fit
against `/assets/profile.md`.

## Verdict

For every hard rule, in the order the rules file lists them, return one row:

| Rule | Result | Quote |

- **pass** - the facts show the rule is met, and the quote shows it.
- **fail** - the facts show the rule is broken, and the quote shows it.
- **unknown** - the posting does not say. A rule with no quote is never a pass.

Then the verdict:

- any **fail** -> `SKIP`, naming the first failing rule as the deciding rule;
- no fail, but an **unknown** on a rule marked _decisive_ -> `ASK`, naming what
  the candidate must find out before applying;
- otherwise -> `APPLY`.

When the headline and the body disagree on experience, apply the rule as
`/assets/rules.md` says to, and say which number decided it.

## Fit score

Score 0-10 from how many of the posting's required skills appear in the
profile, weighting the ones the posting calls required over nice-to-have. List
the matched skills and the missing ones. The score never changes the verdict -
a 9 that breaks a hard rule is still SKIP.

Return the rule table, the verdict line, the deciding rule and the fit section
in your answer; nothing you read is visible to the agent that forked you.
