# Rules

What the candidate applies to and what they do not. `extractor` reports the
Fields; `screener` applies the Hard rules in the order listed.

## Fields

| Field | What to report |
| --- | --- |
| company | Name of the hiring company (not the agency posting for it) |
| posted_by_agency | yes, with the agency name, if a recruiter or staffing firm posts on another company's behalf |
| company_type | product, services (IT services / outsourcing / consultancy / agency building for clients), or not stated |
| role | Job title as written |
| seniority_words | Any of: senior, sr, lead, staff, principal, II, III, manager, architect - as written in the title |
| location | City or cities; remote if stated |
| min_years_headline | Lower bound of the experience range in the header or summary |
| min_years_body | Every other "N years" / "N+ years" requirement in the body, each quoted |
| employment_type | full-time, contract, freelance, internship - as worded |
| duration | Any fixed term (e.g. "3 months") |
| salary | Range in LPA (convert monthly x12, lakhs as written) |
| shift | Working hours or shift as stated (e.g. "1 PM - 10 PM IST", "night shift") |
| work_days | Days per week if stated |
| primary_language | The language the role is built around, if one dominates (e.g. "Java/Spring Boot only") |
| domain | ML research / model training, classical ML, computer vision, app engineering, GenAI application, frontend, backend, etc. |
| skills_required | Skills the posting calls required |
| skills_nice | Skills the posting calls nice-to-have |

## Hard rules

1. **Experience** (_decisive_). Pass when `min_years_headline` is 0 or 1. Fail
   when it is 2 or more. When the headline is 0-1 but the body asks for 2+,
   the headline decides (pass), and the report notes the body number. When
   there is no headline, the lowest body number decides.
2. **Seniority.** Fail if `seniority_words` has any of senior, sr, lead, staff,
   principal, II, III, manager, architect.
3. **Employment type** (_decisive_). Fail for contract, freelance, internship,
   or any fixed `duration`. Unknown when not stated.
4. **Company type.** Fail for services firms and for postings by a staffing
   agency on another company's behalf.
5. **Location.** Pass for a tier-1 Indian city (Bengaluru, Hyderabad, Pune,
   Mumbai / Navi Mumbai, Chennai, Delhi NCR incl. Gurugram and Noida) or remote
   from India. Fail otherwise.
6. **Pay floor.** Fail if a stated salary's upper bound is below 14 LPA. Unknown
   (not decisive) when no salary is stated.
7. **Shift.** Fail for night shifts or any shift ending after 10:30 PM IST.
8. **Work week.** Fail for six-day weeks.
9. **Language.** Fail when the role is Java-only, .NET/C#-only or PHP-only.
10. **Domain.** Fail for classical ML, model training or research-only roles,
    and computer vision. Pass for software engineering and GenAI application
    work.

## Verdict

- Any fail -> SKIP.
- No fail, an unknown on a _decisive_ rule -> ASK.
- Otherwise -> APPLY.
