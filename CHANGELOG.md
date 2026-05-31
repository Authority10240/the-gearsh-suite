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
| 2026-05-31 | TICKET-110..114 | mobile | (no remote yet xe2x80x94 landed directly on main @ c9a1272) | MERGED | Discovery module: DiscoverPage with gradient hero + category row + featured carousel, SearchPage with debounced autocomplete + grid results, ArtistPage with collapsing cover + avatar/rating/verified header + Book CTA + Services/About/Portfolio tabs. DataRemoteDataSource over /v1/categories /v1/artists /v1/services /v1/search /v1/search/autocomplete. ArtistCard widget. 3 factory blocs (DiscoverBloc + SearchBloc + ArtistBloc). Also: DI hotfix for ProfileBloc/EditProfileBloc/SessionsBloc which were referenced but not registered in the prior commit (would have crashed at runtime). flutter analyze 0 issues. |
| 2026-05-31 | TICKET-105..108,153 | mobile | (no remote yet — merged locally to main via --no-ff of feat/profile-settings-help-module @ a6849e6) | MERGED | Profile + settings + help module: ProfilePage with avatar/name/email/role chips + delete-account, EditProfilePage, SessionsPage (list + revoke per /v1/users/me/sessions), SettingsPage hub, LocalePickerPage RadioGroup over 8 locales, NotificationPreferencesPage stub, HelpPage with 5 FAQs + mailto support, AboutPage with version. AuthRepository extended with deleteAccount + listMySessions + revokeMySession. 3 new factory blocs (ProfileBloc, EditProfileBloc, SessionsBloc). 6 new routes regenerated. flutter analyze 0 issues; 13/13 tests passing. |
| 2026-05-31 | TICKET-101..104 | mobile | (no remote yet — merged locally to main via --no-ff of feat/auth-module @ 6d45e84) | MERGED | Auth module: onboarding 3-slide PageView + has-seen-flag deep-link, login form (email + Google + Apple + Huawei via stub FirebaseGateway → /v1/auth/login + /social), register form (email + phone + password), forgot-password, account-security (linked providers + change-password + sign-out / sign-out-everywhere). ProfileStore replaces ProfileEntity singleton. AuthGuard absorbed into app_router with typed LoginRoute. Refresh-interceptor fixed to use AuthSession.expiresIn. Shared validators + WTextField/WButton. 12 use cases + 4 factory blocs registered in DI. 13 tests passing. |
| 2026-05-31 | TICKET-100 | mobile | gearsh-mobile/.tickets/TICKET-100.md (no remote yet — merged locally to main via --no-ff of feat/TICKET-100-scaffold @ 6221fb7) | MERGED | Mobile scaffold: Flutter project, BLoC base classes, Dio per-engine clients with auth+refresh interceptors, Content Engine bootstrap (hydrate + theme builder + context.copy + AppLocalizationsDelegate), stubbed FirebaseGateway, auto_route with AuthGuard/RoleGuard, 15 feature module stubs, 8 ARB locales, dev/staging/prod env files. iOS 14 deployment target. PushService started at boot. flutter analyze 0 issues; tests pass. Verified live on iPhone 16 sim. |
