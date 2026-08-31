# Mobile Integration Plan — closing the 10 outstanding SDK/API integrations

Ordered plan (item 1 → item 10, per product decision 2026-08-25) to take `gearsh-mobile`
from stubbed integrations to real ones. Each item lists the ticket, what gets built,
what access it needs (and from whom), and how it's verified. Delivery follows
[ORCHESTRATION.md](ORCHESTRATION.md): one ticket = one branch = one PR + CHANGELOG row.

**The DI seam already exists**: `injection.dart` binds `StubFirebaseGateway`
unconditionally today; `AppConfig.firebaseProjectId` / `googleMapsApiKey` are empty in
`config.dev.env` by design. The rule everywhere below: **real adapter when the key is
present, stub when it isn't** — so local dev (and the TICKET-161 e2e) keeps working
without credentials forever.

---

## 0. Access requests — fire these first (long lead times)

| # | Access | Owner | Blocks | Lead time | Status |
|---|---|---|---|---|---|
| A1 | **Apple Developer Program** enrolment ($99/yr) | Collins (CLIENT) | Item 1 (Sign in with Apple), Item 2 (APNs push), TestFlight | days → 2 weeks (D‑U‑N‑S) | REQUESTED |
| A2 | **Huawei AppGallery Connect** developer account (free) | Collins (CLIENT) | Item 1 (Huawei sign-in) | 1–2 days identity check | REQUESTED |
| A3 | **GCP projects + billing** (staging/prod) | Collins (CLIENT) | Maps billing (Item 9) — Firebase already live on gearsh-806df | same day | REQUESTED |
| A4 | **Firebase project** (staging + prod), Auth providers enabled | Freedom (ENG) | Items 1, 2, 3 | same day | **VERIFIED** — gearsh-806df live: mobile apps registered, email+Google+Apple providers on, Admin SDK key issued, login e2e green on the real gateway (TICKET-174) |
| A5 | **APNs auth key (.p8)** uploaded to FCM | Freedom (ENG), after A1 | Item 2 on iOS | same day | — |
| A6 | **Apple Services ID + Sign-in key** | Freedom (ENG), after A1 | Item 1 (Apple leg) | same day | — |
| A7 | **Huawei app ID + secret → Firebase custom OIDC** | Freedom (ENG), after A2 | Item 1 (Huawei leg) | same day | — |
| A8 | **Google Maps API key** (iOS + Android SDKs, billing on) | Freedom (ENG), after A3 | Item 9 | same day | — |

Items **4–8 and 10 need no external access** — they run against local engines and can
be built while the requests above are in flight. A dev Firebase project on the company
Google account can unblock Items 1–3 (email + Google legs) the same day, ahead of A3.

---

## 1. Real social sign-in — TICKET-162 (core + Google), TICKET-163 (Apple), TICKET-164 (Huawei)

- **Build**: `RealFirebaseGateway` implementing the existing `FirebaseGateway`
  interface; DI picks real vs stub on `firebaseProjectId.isNotEmpty`. `flutterfire
  configure` → `firebase_options.dart`. Google: `google_sign_in` → Firebase credential →
  ID token → existing `POST /v1/auth/social {provider: GOOGLE}`. Apple: `sign_in_with_apple`
  + Xcode capability + nonce flow. Huawei: **add `huawei_account` to pubspec (currently
  missing)**, AGConnect config, exchange via Firebase custom OIDC; Android/Harmony builds only.
- **Backend**: none — the Auth Engine already verifies real Firebase ID tokens via
  `firebase-admin`; set `FIREBASE_*` env (Secret Manager) and the dev bypass stays
  non-production-only.
- **Access**: A4 (now-ish); A6 and A7 gate the Apple/Huawei legs — those sub-tickets
  land whenever enrolment completes, without blocking Item 2.
- **Verify**: real Google sign-in on simulator against staging Auth Engine;
  TICKET-161 e2e keeps guarding the stub path.

## 2. Push notifications (FCM) — TICKET-165

- **Build**: messaging in `RealFirebaseGateway`: permission prompt, APNs registration,
  token → existing `POST /v1/notifications/devices`; refresh → re-register; revoke on
  logout. Silent push `{contentVersion}` → `ContentBootstrapper` dirty flag (the
  `PushService` wiring already exists). iOS capabilities: Push Notifications +
  Background Modes (remote-notification).
- **Access**: A4 + A5 (iOS). Android works with A4 alone.
- **Verify**: test push from Firebase console; booking-event push from the Data Engine
  dispatcher lands on the device; `device_tokens` row visible in admin.

## 3. Analytics + Crashlytics — TICKET-166

- **Build**: `firebase_analytics`/`firebase_crashlytics` behind the same gateway
  (`logEvent`, `setUserId`, `setCurrentScreen`); `FlutterError.onError` +
  `PlatformDispatcher.onError` hooks; dSYM upload in CI.
- **Access**: A4 only. **Verify**: DebugView stream + a forced test crash in console.

## 4. Media uploads (Media Engine client) — TICKET-167

- **Build**: `core/media` client: `POST /v1/media/upload-sessions` → resumable PUT to
  the signed GCS URL (Dio, chunked, pause/resume) → poll item until `READY` → hand back
  `mediaId`. `image_picker` + photo/camera permission strings. Shared upload widget
  (progress, retry).
- **Access**: none for the code; a dev GCS bucket (ENG) if local GCS emulation proves
  insufficient. **Verify**: avatar upload end-to-end → variant thumbnails resolve.

## 5. Artist self-service — TICKET-168

- **Build**: edit profile (`PUT /v1/artists/me`), portfolio manager (Item 4 uploads +
  `POST /v1/artists/me/portfolio`), availability editor (`POST /v1/artists/me/availability`),
  verification submission (document uploads → `POST /v1/artists/me/verification`).
- **Verify**: submit verification in the app → approve it in the admin dashboard's
  verification queue (full cross-client loop). **Access**: none.

## 6. Services CRUD — TICKET-169

- **Build**: "My services" screen: list / create / edit / deactivate via `/v1/services`
  (+ inclusions). **Verify**: created service appears in Discover + booking flow. **Access**: none.

## 7. Notification preferences — TICKET-170

- **Build**: replace the static stub page with `GET /v1/notifications/preferences` →
  toggles (push/email, booking/messages/marketing) + quiet-hours pickers → `PUT`.
- **Verify**: toggle persists across restart; marketing off suppresses broadcast
  delivery in the notification log. **Access**: none.

## 8. Check-in geolocation — TICKET-171

- **Build**: `geolocator` + permission strings; capture position at check-in and pass
  the `location` the datasource already accepts; graceful denied-permission fallback
  (check-in still works, unstamped).
- **Verify**: booking status history shows the stamp in admin. **Access**: none.

## 9. Map view — TICKET-172

- **Build**: `google_maps_flutter` map mode on Discover/Search; markers from
  `/v1/search` with location filter; `/v1/map/geocode` for place lookup; key injected
  from `GOOGLE_MAPS_API_KEY` (falls back to list view when empty — the config already
  models this). iOS AppDelegate + Android manifest key entries.
- **Access**: A8. **Verify**: map renders artists around a geocoded location.

## 10. Real i18n — TICKET-173

- **Build**: real per-locale ARB files + `intl_utils` codegen replacing the
  same-strings-for-every-locale stub; locale resolution honours the in-app picker;
  Content Engine remains the source for screen copy, ARB covers fixed chrome.
- **Decision needed (product)**: the stub's language list (en/zu/af/pt/fr/sw) vs the
  spec's 8 regional `en-*` locales vs Content Engine locales — reconcile before codegen.
- **Access**: none. **Verify**: switching locale changes chrome strings; RTL-safe.

---

## Sequencing

- **Now, in parallel with access requests**: TICKET-162 scaffolding (gateway class +
  DI switch compiles with stub fallback), then Items 4 → 8 and 10 against local engines.
- **On A4**: finish 162 (Google leg), then 165, 166.
- **On A6 / A7**: 163, 164 (slot in whenever enrolment completes).
- **On A8**: 172.
- Every ticket: `flutter analyze` clean + tests + PR + CHANGELOG row, per ORCHESTRATION.
