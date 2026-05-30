# Changelog

This file is the master PR log. **Agents push their PR link into this file directly on `main`** — this is the only file in the repo for which direct `main` commits are allowed, and only for appending a new row or updating the status of an existing row.

Agents append one row per PR on open and update the status on merge.

**Format** (one Markdown table row, newest at the top of the table):

```
| <date ISO 8601> | <ticket-id> | <engine> | <PR url> | <status> | <one-line summary> |
```

**Status values:** `OPEN`, `CHANGES_REQUESTED`, `MERGED`, `ABANDONED`.

**Engines:** `contracts-package`, `auth`, `data`, `payments`, `content`, `mobile`, `admin-web`, `infra`.

CI enforces the format. Rows that don't match are rejected.

---

## Entries

| Date | Ticket | Engine | PR | Status | Summary |
|------|--------|--------|-----|--------|---------|
<!-- agents append new rows directly below this line, newest first -->
| 2026-05-27 | TICKET-100 | mobile | gearsh-mobile/.tickets/TICKET-100.md (no remote yet — local feat/TICKET-100-scaffold) | OPEN | Mobile scaffold: Flutter project, BLoC base classes, Dio per-engine clients with auth+refresh interceptors, Content Engine bootstrap (hydrate + theme builder + context.copy), stubbed FirebaseGateway, auto_route with AuthGuard/RoleGuard, 15 feature module stubs, 8 ARB locales, dev/staging/prod env files. flutter analyze 0 issues; tests pass. |
