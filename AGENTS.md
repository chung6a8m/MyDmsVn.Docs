# Working in MyDmsVn.Docs

This repository owns analysis, cross-repository design proposals and decisions. Product source, executable skills, actual test commands and release evidence stay with their owning repositories unless an explicit migration moves them.

## Writing

- Write user-facing documents in Vietnamese, preserving identifiers and diacritics.
- Read README and the specific document being changed. Do not load every historical report by default.
- Keep observed source facts, recorded runtime evidence, recommendations and approved decisions distinct.
- Cite code with repository, exact commit and path; include line anchors where useful. Do not substitute mutable upstream HEAD for a consumed dependency pin.
- Keep decision status in the adoption document. Link to it instead of copying progress into every page.
- A committed proposal is not approval to implement it or to reactivate a retired plan.
- Do not copy credentials, connection strings, private repository content or raw runtime data into this public documentation repository.

## Verification

- For Markdown-only work, check diff, local links/anchors, status consistency and UTF-8 encoding. SQL/GUI/build execution is not needed just to edit prose.
- Mark benchmarks and future speed budgets as proposed unless actually measured.
- Preserve unresolved acceptance limits when summarizing evidence. Historical PASS does not establish a new candidate PASS.
- Follow .editorconfig and .gitattributes. Keep Git text normalized and Markdown UTF-8 without BOM.

## Scope

Do not turn documentation maintenance into another test runner, approval-file system or parallel active roadmap. When a proposal changes, update its owner document and repair incoming links.
