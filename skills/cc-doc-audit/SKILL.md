---
name: cc-doc-audit
description: >-
  Audits project and agent documentation for contradictions, code drift, and
  dead docs/paths, then applies fixes only after explicit user approval of the
  full fix package. Use when the user invokes /doc-audit or explicitly asks to
  audit docs, check documentation drift, or find doc contradictions.
disable-model-invocation: true
---

# doc-audit

Audit agent + product docs for contradictions, code drift, and dead paths. Report in Turkish. Edit only after the user approves the full fix package.

## Scope

### Include

**Agent docs**

- `AGENTS.md`, `CLAUDE.md`
- `.cursor/rules/**`, `.cursor/skills/**`
- Repo-local `.agents/skills/**`

**Product docs**

- `README*`
- `CHANGELOG*`, `HISTORY.md`, `CHANGES.md`
- `CONTRIBUTING*`
- `SECURITY.md`, `CODE_OF_CONDUCT.md`, `SUPPORT.md`, `GOVERNANCE.md`
- `docs/**`
- `**/ADR*`, `**/adr/**`
- PRD / spec markdown under common paths (`**/prd/**`, `**/specs/**`, `**/*PRD*.md`, `**/*SPEC*.md`)
- Repo root / `.github` community docs that assert product facts (e.g. `.github/PULL_REQUEST_TEMPLATE*`, `.github/ISSUE_TEMPLATE/**`)
### Skip

- `node_modules/**`, build outputs, lockfiles
- Generated API dumps unless a tracked doc links to them

## Finding types (only these)

| Type | Name            | Meaning                                          |
| ---- | --------------- | ------------------------------------------------ |
| 1    | doc↔doc         | Two docs disagree on the same fact               |
| 2    | doc↔code drift  | Doc does not match actual code behavior          |
| 3    | —               | **Out of scope** (missing docs)                  |
| 4    | dead doc / path | Doc or link/path points at missing targets       |
| 5    | —               | **Out of scope** (terminology → domain-modeling) |

## Source of truth

- **Type 2:** code wins. Propose doc edits only. Do not change code unless the user explicitly overrides.
- **Type 1:** ask which doc wins before editing.
- **Type 4:** fix broken references/paths in the package. Never delete files in the package — ask separately for deletes.

## Workflow

### 1. Audit (read-only)

1. Inventory in-scope docs (glob + targeted search). Prefer context-mode / targeted reads over dumping whole trees.
2. Cross-check claims across docs and against code for types 1, 2, 4 only.
3. Attach evidence as `path:line` (or path + short quote).
4. Collapse duplicates. Cap at ~15 findings; if truncated, say so.
5. Emit the Turkish report (template below). **Stop. No edits.**

### 2. Gate

Propose **one** fix package covering all findings.

Wait for explicit approval: `uygula` / `apply` / equivalent.

Do not edit on “looks good” alone unless it clearly means apply the package.

### 3. Fix (after approval)

1. For any Type 1 still unresolved on winner: ask once, then edit.
2. Apply the entire approved package.
3. Type 4: update or remove broken refs only; never delete doc files without a separate ask.
4. Summarize what changed (paths). Offer `kaydet` if not already saved.

### 4. Optional save

On `kaydet` / `save`: write the report to `docs/_audit/YYYY-MM-DD-doc-audit.md` (create `docs/_audit/` if needed).

## Turkish report template

Lead with the next action. Keep it ADHD-short.

```markdown
**Sonraki:** `uygula` | `kaydet` | bulgu yok

## Özet

- Tip 1 (doc↔doc): N
- Tip 2 (drift): N
- Tip 4 (ölü/path): N

## Bulgular

| ID  | Tip   | Kanıt         | Önerilen fix |
| --- | ----- | ------------- | ------------ |
| F1  | 1/2/4 | `path:line` … | …            |

## Fix paketi

- [ ] F1: …
- [ ] F2: …

Decision: PASS | FAIL

**Sonraki:** …
```

- `PASS` = no findings (or only intentionally deferred).
- `FAIL` = one or more findings in the package.
- End with one concrete next action (`uygula`, `kaydet`, or stop).

## Hard rules

1. No file edits before explicit package approval.
2. Explicit invoke only (`/doc-audit` or user attaches / names this skill). Do not auto-start from ambient chat.
3. Do not hunt missing docs (type 3) or terminology (type 5).
4. Do not rewrite docs for style/voice.
5. Do not commit or open PRs unless the user asks separately.
6. Prefer evidence over vibes: every finding needs a citation.

## Examples

**User:** `/doc-audit`

→ Inventory → findings F1… → Turkish report → stop at gate.

**User:** `uygula`

→ Apply full package; ask Type 1 winners if needed; no file deletes.

**User:** `kaydet`

→ Write `docs/_audit/YYYY-MM-DD-doc-audit.md`.
