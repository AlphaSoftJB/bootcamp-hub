# Learner Data Inventory

**Version:** 1.0  
**Date:** 2026-11-05  
**Status:** Draft — extracted from compiled bundle; pending validation against live browser storage  
**Author:** Sprint 1 bundle analysis (Claude Sonnet 4.6, session https://claude.ai/code/session_01GrXQBHN1FfE3b1PuBWsGTt)  
**Purpose:** Satisfies DR-019 precondition #1 — full inventory of every localStorage key used for progress before GistSync or any localStorage key may be removed.

> **Source:** All keys and formats below were reverse-engineered from the compiled bundle  
> `assets/index-u5-2w4C2.js` (loaded by `index.html` at commit `138d6e7`). The source  
> code has never been committed to the repository; this is the authoritative extraction.  
> Validation against live browser storage is a required next step (see §7).

---

## 1. Summary

| Category | Keys | Removable in Phase 1? |
|---|---|---|
| XP / Gamification | 1 key | No — migrate to backend first |
| Lab / lesson completion | Dynamic (N keys) | No — migrate to backend first |
| PBQ / drill scores | Dynamic (N keys) | No — migrate to backend first |
| GistSync credentials | 2 keys | No — remove per DR-020 gate only |
| UI preferences | 2 keys | Yes — these are non-progress |
| Notes / project work | Dynamic (N keys) | No — migrate to backend first |

---

## 2. Progress Keys (must NOT be removed before migration)

### 2.1 XP and Gamification

**Key:** `bootcamp_xp_v1`  
**Constant:** `Uu` (minified)  
**Type:** JSON object  
**Format:**
```json
{
  "totalXP": 0,
  "streak": 0,
  "lastActiveDate": null,
  "achievements": []
}
```
**Notes:** Read by `ty()` function at app load. This is the primary gamification state. `achievements` is an array — format of individual achievement objects TBD (requires live storage sample).

---

### 2.2 Lab / Lesson Completion Flags

Three distinct key patterns exist for marking a lab or lesson day complete. All store the string `"1"` as the value.

**Pattern A — Generic lab completion:**  
Key: `lab_done_${id}` where `id` = module or lesson identifier  
Example: `lab_done_42`, `lab_done_intro-linux`

**Pattern B — Cyber-specific lab completion:**  
Key: `lab_done_cyber-${e.id}` where `e.id` = cyber module ID  
Example: `lab_done_cyber-net-basics`

**Pattern C — Module + day completion:**  
Key: `lab_done_${moduleId}-${day}` where `moduleId` = parent module ID, `day` = day number  
Example: `lab_done_py-ml-1-3` (module `py-ml-1`, day `3`)

**Pattern D — PBQ / drill lab key:**  
Key: `lab_done_${labKey}` where `labKey` = PBQ drill compound key  
Example: `lab_done_pbq-net-002`

> **Important:** The exact ID namespaces (what values appear in `id`, `moduleId`, `day`) require  
> inspection of the data files (e.g., `cyberLabsNew.js`, IBM Python/ML data files). These are  
> compiled into the bundle. Full enumeration of all actual stored keys requires a live browser  
> storage dump from an active learner session.

---

### 2.3 PBQ / Practice Drill Scores

**Key pattern:** `pbq_score_${e}` (returned by function `Qu(e)`)  
**Constant:** `Qu` (minified function name)  
**Type:** JSON object  
**Format:**
```json
{
  "got": 0,
  "total": 0
}
```
Where `got` = questions answered correctly, `total` = total questions in the drill.  
`e` = the PBQ drill identifier (e.g., `"net-001"`, `"sec-firewall"` — exact values require data file inspection).

---

### 2.4 Lab Notes / Project Reflections

**Key pattern:** Computed key (minified — exact function not yet isolated)  
**Type:** JSON object  
**Format:**
```json
{
  "code": "",
  "reflection": ""
}
```
Where `code` = learner's code or lab work, `reflection` = learner's written reflection.  
**Notes:** These store actual learner-authored text content, not just completion flags. Loss of this data is not acceptable.

> **Action required:** Isolate the key-generating function from the bundle to determine the exact  
> key format for notes storage. This is blocked until source code is available or further bundle  
> analysis is performed.

---

## 3. GistSync Keys (must NOT be removed before DR-020 gate)

**Key 1:** `gist_token`  
**Constant:** `Ds` (minified)  
**Type:** String  
**Format:** GitHub Personal Access Token (PAT) in plaintext  
**Security note:** PAT stored in `localStorage` is accessible to any JavaScript on the page. This is a known security limitation of the current implementation. DR-020 specifies this must be removed — but only after the full DR-020 gate is satisfied.

**Key 2:** `gist_id`  
**Constant:** `Es` (minified)  
**Type:** String  
**Format:** GitHub Gist ID (e.g., `"a1b2c3d4e5f6..."`)  
**Notes:** The gist ID identifies the specific GitHub Gist used for cross-device sync. Loss of this key means the user loses their sync connection and would need to re-link.

---

## 4. UI Preference Keys (safe to migrate to Zustand + localStorage independently)

These keys store non-progress UI preferences. They do not contain learner progress data and may be migrated to the new SPA's Zustand store independently of the backend migration.

**Key:** `bh_theme`  
**Type:** String  
**Values:** `"dark"` | `"light"`  
**Default behavior:** App defaults to dark if key is absent.  
**Notes:** The bundle contains logic that removes stale values (`"sm"`, `"md"`, `"lg"` seen in this key indicate a schema version mismatch — the app cleans these up on load).

**Key:** `bh_fontSize`  
**Type:** String  
**Values:** `"sm"` | `"md"` | `"lg"` (font size preference)  
**Notes:** Stale values (`"sm"`, `"md"`, `"lg"` stored in `bh_theme`) are cleaned by checking this key — the app self-repairs on load if the wrong value type is found.

---

## 5. Keys Requiring Further Investigation

The following were identified from bundle context but require further analysis:

| Key/Pattern | Status | Note |
|---|---|---|
| Notes/project key | Unknown exact format | Key-generating function not yet isolated; stores `{code, reflection}` |
| Achievement objects in `bootcamp_xp_v1.achievements` | Partial | Array format confirmed; individual object schema unknown |
| `idle` | Unclear | Appears as a state string; may not be a standalone localStorage key |

---

## 6. Data Formats for Migration Planning

### 6.1 Backend schema implications

| localStorage Key | Backend target | Migration action |
|---|---|---|
| `bootcamp_xp_v1` | `user_xp` table (or `progress_snapshots`) | Export, transform, insert on first login |
| `lab_done_*` | `lesson_completions` table | Enumerate all keys, map to content IDs, insert per user |
| `pbq_score_*` | `assessment_attempts` table | Export scores, map PBQ IDs to assessment IDs |
| Notes/reflections | `lab_submissions` or `notes` table | Export text content, preserve per user |
| `gist_token` | Deleted — not migrated | Remove after DR-020 gate |
| `gist_id` | Deleted — not migrated | Remove after DR-020 gate |
| `bh_theme` | UI preference in Zustand | Carry forward in new SPA localStorage |
| `bh_fontSize` | UI preference in Zustand | Carry forward in new SPA localStorage |

---

## 7. Required Validation Steps

Before GistSync or any progress key may be removed (DR-019 precondition #1 is this document; preconditions #2 and #3 require the following):

- [ ] **Live storage dump:** Export actual localStorage contents from a browser session with real learner data. Confirm all key patterns above against real data.
- [ ] **Enumerate all `lab_done_*` keys:** A live session will show every actual `lab_done_*` key name, confirming the namespace values used by the data files.
- [ ] **Identify notes key format:** Inspect data with real lab work completed to find the exact key name for `{code, reflection}` storage.
- [ ] **Enumerate `achievements` array:** Confirm the shape of individual achievement objects in `bootcamp_xp_v1`.
- [ ] **Source code review:** Once source is available (see §8), confirm all key patterns against `src/context/` files.

---

## 8. Source Code Status

> **Critical:** The React source code (`src/`, `vite.config.ts`, `package.json`, component files,  
> context providers, data files) has never been committed to the `AlphaSoftJB/bootcamp-hub`  
> repository in its 47-commit history. The repository contains only compiled build output.
>
> This inventory was produced entirely from bundle analysis. While it is believed to be complete  
> and accurate, **source code review would confirm and may reveal additional keys or patterns.**
>
> Sprint 1 implementation is currently blocked pending confirmation of source code location  
> (see Sprint 1 blocker report).

---

## 9. Export Tool Specification

DR-019 precondition #2 requires a learner data export tool. The tool must:

1. Read all keys identified in §2 and §3 from the user's browser `localStorage`
2. Output a JSON file with structure:
   ```json
   {
     "exported_at": "ISO8601 timestamp",
     "xp": { "totalXP": 0, "streak": 0, "lastActiveDate": null, "achievements": [] },
     "completions": { "lab_done_key_1": "1", "lab_done_key_2": "1" },
     "pbq_scores": { "pbq_score_net-001": { "got": 0, "total": 0 } },
     "notes": { "<notes_key>": { "code": "", "reflection": "" } },
     "gist_id": "...",
     "exported_app_version": "bootcamp_xp_v1"
   }
   ```
3. Provide this file for the user to save before any migration or key removal
4. Must NOT include `gist_token` in export (security — PAT must not be exported)

**Implementation:** A small standalone HTML page or browser console snippet is sufficient. Full backend integration not required for the export tool — the export is a one-time migration aid.

---

*This document satisfies DR-019 precondition #1. See DR-019 and DR-020 in the Decision Record for the full gate requirements before GistSync removal may proceed.*
