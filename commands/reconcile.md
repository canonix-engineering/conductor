---
description: Reconciles track spec and plan with codebase changes and fixes inconsistencies
---

## 1.0 SYSTEM DIRECTIVE
You are an AI agent assistant for the Conductor spec-driven development framework. Your current task is to analyze the current track's specification and plan documents for inconsistencies, dead paths, and drift from codebase changes since the spec was written.

CRITICAL: You must validate the success of every tool call. If any tool call fails, you MUST halt the current operation immediately, announce the failure to the user, and await further instructions.

## 1.1 SETUP CHECK
**PROTOCOL: Verify that a track is active and properly configured.**

1.  **Check for Active Track:** Look for non-archived tracks in `conductor/tracks/`. Identify the current track by:
    -   Checking git branch name for track hints
    -   Finding tracks with `status: "in_progress"` in metadata.json
    -   If multiple candidates, ask user which track to reconcile

2.  **Load Track Files:** Read:
    -   `conductor/tracks/<track_id>/spec.md`
    -   `conductor/tracks/<track_id>/plan.md`
    -   `conductor/tracks/<track_id>/metadata.json`

3.  **Handle Missing Files:**
    -   If track files missing, halt and announce: "Track files not found. Please verify the track exists."

---

## 2.0 CODE DRIFT CHECK (CONDITIONAL)
**PROTOCOL: Compare codebase changes since spec was written.**

### 2.1 Check for base_sha

1.  **Read metadata.json:** Look for `base_sha` field.

2.  **If `base_sha` exists:**
    -   Get current HEAD SHA: `git rev-parse HEAD`
    -   Announce: "Checking for code changes since spec baseline ({base_sha})..."
    -   Proceed to 2.2

3.  **If `base_sha` missing:**
    -   Announce: "Code drift check skipped: no `base_sha` in metadata.json. Only checking document consistency."
    -   Skip to Section 3.0

### 2.2 Get Changed Files

1.  **Run git diff:**
    ```bash
    git diff {base_sha}..HEAD --name-only
    ```

2.  **Filter to relevant paths:**
    -   `app/models/**/*.rb`
    -   `app/controllers/**/*.rb`
    -   `app/services/**/*.rb`
    -   `app/concerns/**/*.rb`
    -   `lib/tasks/**/*.rake`
    -   `config/routes.rb`
    -   `db/migrate/*.rb`
    -   `Gemfile`

3.  **If no relevant changes:** Announce "No relevant code changes since spec baseline." and proceed to Section 3.0.

### 2.3 Analyze Changed Files for Impact

For each changed file, determine if it affects the spec:

**Model changes:**
-   New/removed columns that spec references
-   Changed associations
-   New validations or callbacks
-   Renamed models

**Controller changes:**
-   New/changed actions that spec references
-   Authentication/authorization changes
-   Response format changes

**Service changes:**
-   Interface changes to services spec plans to use
-   New services that could be reused
-   Removed services spec depends on

**Migration changes:**
-   Schema changes affecting spec's data model assumptions

**Route changes:**
-   Endpoint paths that spec references

**Gem changes:**
-   Dependencies spec assumes exist
-   New gems that affect approach

**Rake task changes:**
-   Tasks that spec references or plans to create

---

## 3.0 DOCUMENT CONSISTENCY CHECK
**PROTOCOL: Analyze spec and plan for internal issues.**

### 3.1 Check for Inconsistencies

Scan for:
-   Cross-references between FRs that don't match (e.g., "see FR-1.4" pointing to wrong content)
-   Conflicting statements about the same feature in different sections
-   Plan tasks that contradict spec requirements
-   Acceptance criteria that don't match functional requirements
-   Architecture section components not matching FRs

### 3.2 Check for Dead Paths

Scan for:
-   FRs that reference removed/changed approaches
-   Plan tasks for features that were descoped or changed
-   Redundant FRs that duplicate information without adding value
-   Out-of-scope items that contradict in-scope items
-   Leftover language from previous design iterations (e.g., "admin creates" when automated)

### 3.3 Check for Orphaned References

Scan for:
-   References to non-existent FRs
-   Plan phases referencing removed spec sections
-   Acceptance criteria for removed features

---

## 4.0 REPORT FINDINGS
**PROTOCOL: Present findings in structured format.**

Generate report:

```markdown
## Reconciliation Report for {track_id}

### Git Context
- Spec baseline: {base_sha or "not set"}
- Current HEAD: {current_sha}
- Code drift check: {enabled | skipped (no base_sha)}

### Summary
- Code drift issues: {N}
- Document inconsistencies: {N}
- Dead paths: {N}
- Total issues: {N}

### Code Drift (from main)

| # | File | Change | Spec Impact | Suggested Fix |
|---|------|--------|-------------|---------------|
| 1 | ... | ... | ... | ... |

*(Or "Skipped: add `base_sha` to metadata.json to enable")*

### Document Inconsistencies

| # | Location | Issue | Suggested Fix |
|---|----------|-------|---------------|
| 1 | ... | ... | ... |

### Dead Paths

| # | Location | Issue | Suggested Fix |
|---|----------|-------|---------------|
| 1 | ... | ... | ... |

### Recommendation

{Summary of most critical issues and recommended action}
```

---

## 5.0 USER CONFIRMATION AND FIXES
**PROTOCOL: Get approval before making changes.**

### 5.1 Present Options

After presenting report, ask:
> "I found {N} issues. How would you like to proceed?"
> - A) Fix all issues
> - B) Review and select specific fixes
> - C) Skip fixes for now

### 5.2 Apply Fixes (if approved)

1.  **Make edits:** Update `spec.md` and `plan.md` as needed
2.  **Preserve structure:** Maintain document formatting and section order
3.  **Update cross-references:** When renumbering FRs, update all references
4.  **Update metadata.json:**
    -   If spec was significantly regenerated: set `base_sha` to current HEAD
    -   Update `updated_at` timestamp

### 5.3 Announce Completion

> "Reconciliation complete. {N} issues fixed."
> - spec.md: {changes made}
> - plan.md: {changes made}
> - metadata.json: {base_sha updated | unchanged}
>
> "Changes are staged but not committed. Review and commit when ready."

---

## 6.0 METADATA SCHEMA

Track metadata.json should include:

```json
{
  "track_id": "string (required)",
  "type": "feature | bugfix | chore",
  "status": "new | in_progress | completed | archived",
  "created_at": "ISO timestamp",
  "updated_at": "ISO timestamp",
  "description": "string",
  "base_sha": "string (git SHA when spec was created/regenerated, optional)"
}
```

### base_sha Rules

| Event | base_sha Action |
|-------|-----------------|
| `/conductor:newTrack` creates spec | Set to current HEAD |
| `/conductor:reconcile` regenerates spec | Update to current HEAD |
| `/conductor:reconcile` minor fixes only | Keep existing |
| `/conductor:implement` | No change |
| `/conductor:review` | No change |

---

## 7.0 NOTES

-   This command is read-only until user approves fixes
-   Focus on structural/semantic issues, not style or formatting
-   When in doubt, flag for user review rather than auto-fix
-   If no issues found, report "Track documents are consistent with codebase"
-   Pay special attention to: model schemas, service interfaces, authentication patterns, shared concerns
-   Highlight when codebase already has something spec plans to create (avoid duplicate work)
