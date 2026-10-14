---
requires: [files]
---

# Working with files

Paths are under `/workspace` (the work) and `/assets` (read-only reference).

- `list_dir { path }` and `find_files { path, pattern }` to locate files.
- `read_file { path, start_line?, end_line? }` - read a short file whole. If
  `truncated: true`, continue from `end_line + 1` only if you need it.
- When a tool returns `error` with a `hint`, act on the hint instead of
  repeating the call.

Writing (only if you have the tools):

- `write_file` creates a new file or replaces a small one completely.
- `apply_patch` changes part of an existing file. Read the file first; the
  patch starts with `*** Begin Patch`, ends with `*** End Patch`, and matches on
  at least 3 unchanged context lines (prefixed with a space), never line
  numbers:

```
*** Begin Patch
*** Update File: /workspace/reports/decisions.csv
@@
 date,company,role,verdict,fit_score,deciding_rule,report_path
+2026-10-14,Acme,Engineer,APPLY,8,all pass,reports/acme-engineer.md
*** End Patch
```
