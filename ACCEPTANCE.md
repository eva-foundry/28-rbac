# 28-rbac -- Acceptance Criteria

**Created**: 2026-03-03 by agent:copilot
**Last Updated**: 2026-03-03
**Data Model**: GET http://localhost:8010/model/projects/28-rbac
**Status**: All gates PENDING (primed -- not yet executed)

---

<!-- eva-primed-acceptance -->

## Summary

| Gate | Criteria | Status |
|------|----------|--------|
| AC-1 | Data model record exists and is current | PENDING |
| AC-2 | copilot-instructions.md is v3.1.0 compliant (PART 1/2/3 present, ASCII-clean) | PENDING |
| AC-3 | Veritas trust score >= 0.6 (MTI baseline) | PENDING |
| AC-4 | All tests pass (exit 0) | PENDING |
| AC-5 | No non-ASCII characters in any .md or .ps1 or .py file | PENDING |
| AC-6 | PLAN.md has active sprint stories with IDs | PENDING |
| AC-7 | README.md references data-model API and veritas | PENDING |
| AC-8 | (project-specific criteria -- fill in) | PENDING |

---

## AC-1: Data Model Record Current

**Criteria**: `GET /model/projects/28-rbac` returns maturity, phase, pbi_total >= 1.
**Verification**:
```powershell
Invoke-RestMethod "http://localhost:8010/model/projects/28-rbac" |
    Select-Object id, maturity, phase, pbi_total, pbi_done, notes
```
**Status**: PENDING

---

## AC-2: copilot-instructions.md Compliant

**Criteria**: PART 1, PART 2, PART 3 all present. ASCII-clean. No stale v2.x markers.
**Verification**: Run MCP tool `audit_project` with target_path = C:\eva-foundry\28-rbac
**Status**: PENDING

---

## AC-3: Veritas Trust Score

**Criteria**: MTI score >= 0.6 on first audit.
**Verification**: Run MCP tool `get_trust_score` with repo_path = C:\eva-foundry\28-rbac
**Status**: PENDING

---

## AC-4: Tests Pass

**Criteria**: All project tests exit 0.
**Verification**: (fill in project-specific test command)
**Status**: PENDING

---

## AC-5: ASCII Compliance

**Criteria**: No non-ASCII characters in tracked .md, .ps1, .py files.
**Verification**: Run MCP tool `check_encoding` with target_path = C:\eva-foundry\28-rbac
**Status**: PENDING

---

## AC-6: PLAN.md Has Sprint Stories

**Criteria**: PLAN.md contains at least one Story with an ID (pattern `[ID=`).
**Verification**: Manual review of PLAN.md
**Status**: PENDING

---

## AC-7: README References EVA Tools

**Criteria**: README.md contains `<!-- eva-primed -->` sentinel and links to data-model + veritas.
**Verification**: Inspect README.md for sentinel
**Status**: PENDING

---

## AC-8: Project-Specific Criteria

> Replace this gate with criteria specific to this project's deliverables.

**Criteria**: (TBD)
**Verification**: (TBD)
**Status**: PENDING
