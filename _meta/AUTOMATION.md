# Automation runbook

Process the next eight source pages from `_meta/progress.json`. Include one neighboring page on each side and all bookmark entries intersecting the batch.

## Per-run contract

1. Verify source hashes before editing.
2. Extract concepts, relationships, invariants, normative requirements, identifiers, state transitions, and cross-references.
3. Update canonical concept pages rather than creating synonyms or one page per field.
4. Add a useful ASCII diagram, compact Markdown table, or explicit `visual not useful` judgment to each concept page touched.
5. Put exact register/command/data layouts and cross-page tables into `review_queue` until compared with rendered pages.
6. Every substantive statement and every diagram must cite local PDF pages.
7. Update the concept index, relevant maps, and `CONTEXT.md` when vocabulary stabilizes.
8. Run `tools/check_wiki.py nvme-wiki`; advance progress only after checks pass.
9. After the final page, perform link, alias, terminology, diagram, and contradiction passes before setting `status` to `complete`.

## Concept page shape

```markdown
# Canonical concept

Short definition.

## Mental model
ASCII diagram or a short explanation of why no diagram helps.

## Contract and invariants
## Lifecycle or structure
## Relationships
## Edge cases
## Evidence
```
