# Yusr — Pre-publication review

**Keep this file out of the repository.** It lives next to the `yusr/` folder on purpose. No secret values are reproduced here; files and locations are named so you can act on them.

Reviewed: `yusr.zip` (334 entries, ~9 MB, **no `.git` folder**) and `Yusr_Report_Final.pdf` (113 pages).
Not done: building, running, `flutter analyze` or `flutter test` (no Flutter SDK or network in the review environment). Everything below comes from reading the code and the report.

---

## 1. What I verified in the source

**Entry point and architecture.** `lib/main.dart` initialises Firebase (`firebase_options.dart`) and `easy_localization` (Arabic default, English fallback), wraps the app in `MultiProvider` (`ThemeProvider`, `AuthProvider`), and starts at `SplashScreen`. Splash waits for auth state and opens `LoginScreen` or `MainWrapper`. Structure is `lib/core/{constants,models,providers,services,theme}` plus `lib/features/<feature>/`. About 14,200 lines of Dart.

**Navigation.** Plain `Navigator`/`MaterialPageRoute`. `go_router` is declared but never used. `PrayerTimesScreen` and `OnboardingScreen` exist but nothing navigates to them. `MainWrapper` tabs: Home, Map, centre button → ritual guide, Supplications, Profile. Admin area opens from the Profile screen when `role == 'admin'`.

**Backend.** Firestore collections used: `users` (+ subcollections `rituals`, `journey_progress`), `duas`, `alerts`, `sos_requests`, `crowd_data`, `announcements`. `ritual_guides` and `prayer_config` are seeded but never read. External HTTP: AlAdhan (prayer times, Makkah, method 4, plain `http://`) and Open-Meteo (temperature).

**Features that match the report:** email/password + Google auth, Gemini chat (`gemini-3-flash-preview`, Arabic system instruction, in-memory session), supplications (search, categories, favourites, 16 seeded duas), persistent Tawaf counter, SOS create/cancel, admin dashboard/users/alerts/duas/crowd, AR/EN + RTL, light/dark theme.

**Where the supplied code does NOT match the report** (important — this is why the README has a status column):

| Report says | Source shows |
|---|---|
| Sa'i counter, five ritual types in the counter | Counter is hardcoded to `tawafUmrah`; Sa'i checklist items open the same Tawaf screen |
| Journey **notes** (`JourneyNoteModel`, `journey_notes` subcollection, add/edit/delete) | Journey is a static itinerary checklist saved to `journey_progress`; no note model or editor |
| Map reads `map_locations` from Firestore, Directions polyline; admin "Map Locations" screen | Map uses 8 hardcoded points; no Directions call; no `map_locations`; admin has 6 screens, none for map points |
| Security (911) / Ambulance (997) dial via `url_launcher` | Buttons have no handler; `url_launcher` not in `pubspec.yaml` |
| Location-aware prayer times + notifications | Fixed to Makkah; no notification code |
| Real-time location tracking (FR-03) | `LocationTrackingService` is never instantiated |
| FCM, Cloud Storage, Apple sign-in | Not in `pubspec.yaml` or code |
| Admin resolves SOS requests | `resolveSosRequest` exists but no screen calls it (admin can change user status instead) |
| Firestore security rules (Ch. 7.3.4) | No `firestore.rules` in the zip; `firebase.json` has no Firestore section |
| Immersive system UI mode in `main.dart` | Not present |
| "Flutter 3.10.4" | `pubspec.yaml` constrains the **Dart** SDK to `^3.10.4`; the report conflates the two |

The report's UI screenshots show a richer map (many markers) and a login screen with an Apple button, so the **report was probably written against a newer build than this zip** (latest file in the zip is dated 5 Mar 2026; the report is dated June 2026). If you have the final code, use that instead and I can re-check the README against it.

**Other observations.** Profile screen only displays data — there is no UI to enter Hajj ID, blood type or emergency contact, yet SOS reads them. The user-facing alert list uses `streamAlerts` (the admin stream) with no `isActive` filter. Favourites are stored on the shared `duas` document, so they are not per-user. The admin dashboard "seed data" button writes test users and demo data into whatever Firestore project the app points to. Release builds are signed with the debug key; package ID is `com.example.morshedy`; `pubspec.yaml` name is `morshedy`. `test/widget_test.dart` is the untouched Flutter counter template and would fail.

---

## 2. Screenshots

**Extracted** from report pages 61–70 (the original embedded raster images, not page renders, ~400×900 px each), cropped to remove status-bar strips:

| File | Source |
|---|---|
| `home-screen.png`, `home-screen-ar.png` | p. 62 |
| `ai-chatbot.png` | p. 63 |
| `haram-map.png` | p. 64 |
| `safety-alerts.png`, `safety-alerts-ar.png` | p. 65 |
| `sos-confirmation.png` | p. 66 |
| `tawaf-counter.png` | p. 67 |
| `supplications-library.png`, `supplications-library-ar.png` | p. 68 |
| `admin-dashboard.png` | p. 69 |
| `profile-settings.png` | p. 70 |

**Redacted (pixelated + blurred):** the user's name in the home header (both languages — it is a real team member's name), name/email/ID in the profile card, and the pilgrim names in the admin SOS banner. Original captures were not changed on disk.

**Deliberately not used:** the login screens (p. 61) — they show an Apple button that the code does not implement. No report page images were used, so no student IDs, e-mails or signatures are included.

**Still worth capturing from a running build of *this* repository** (the report images are low resolution and from a possibly different build):

1. All eight priority screens again at full device resolution (they would replace the report captures).
2. Login and register screens (no Apple button).
3. Journey checklist and ritual guide.
4. Admin: users, alerts, duas, crowd map.
5. Active-SOS state and the "I'm safe" button.
6. Light theme versions of home and map.
7. Optional: a short GIF of the Tawaf counter or AI chat.

Use a test account with a neutral name, and blur or avoid real names/emails.

---

## 3. Do not commit

| File / folder | Why | Handled in `.gitignore`? |
|---|---|:---:|
| `lib/features/ai_chat/ai_chat_screen.dart` (a **string literal inside tracked source**) | Contains a hardcoded **Gemini API key** — the most serious finding | n/a — edit the code |
| `android/app/src/main/AndroidManifest.xml` | Hardcoded **Google Maps (Android) key** in `com.google.android.geo.API_KEY` | n/a — edit the code |
| `ios/Runner/AppDelegate.swift` | Hardcoded **Google Maps (iOS) key** (same value as the Android Maps key in this snapshot) | n/a — edit the code |
| `android/app/google-services.json` | Firebase project ID, app ID, client API key, OAuth client | yes |
| `lib/firebase_options.dart` | Firebase API keys for web/Android/iOS/macOS/Windows, project ID, iOS client ID | yes |
| `firebase.json` | Project ID and app IDs (identifiers, not secrets) | yes |
| `android/local.properties` | Local SDK paths; Flutter writes it per machine | yes |
| `.idea/`, `*.iml`, `.zencoder/`, `.zenflow/`, `android/.kotlin/`, `devtools_options.yaml` | IDE / tooling metadata, local paths | yes |
| `Yusr_Report_Final.pdf` | Contains student IDs, university e-mails, signatures, plus university IP | not in the repo folder — keep it out |

**Checked and absent:** `GoogleService-Info.plist`, `key.properties`, `*.jks`/`*.keystore`, `.env` files.

**Key inventory (no values shown).** Five distinct `AIza…` keys exist: one Maps key (manifest + AppDelegate), one Gemini key (chat screen), and three Firebase client keys (google-services.json/Android, web+windows, iOS+macOS). Firebase client keys are identifiers rather than true secrets, but the Gemini and Maps keys are billable credentials.

**Decision I made:** I treated the Firebase config files as "do not commit" and told the README to use `flutterfire configure`, because the team's live Firebase project would otherwise be public and I could not see its security rules. The trade-off is that the repo will not compile until a cloner runs that command (documented). If you prefer a zero-setup repo, you can instead commit them **after** restricting the keys and confirming strict Firestore rules — but never the Gemini key.

I did not modify any source files. The zip is not a Git repository, so there is no commit history exposing the keys *within it* — but see below.

---

## 4. Required before publication

1. **Rotate the Gemini key** (create a new one in Google AI Studio, delete the old one) — it was embedded in source and in a zip that has been shared. Do the same for the Maps key.
2. **Restrict** the new Maps keys (Android: package + SHA-1; iOS: bundle ID; limit to Maps SDK APIs) and the Firebase API keys (Google Cloud → Credentials).
3. **Remove the three hardcoded literals** from the files listed in section 3 and load them as described in the README ("Suggested approach"). I did not change code for you.
4. **Check Firestore rules** in the Firebase console for project `yusr-c69f7`. No rules file was supplied. Rules must block non-admins from admin collections and from editing their own `role`/`isBanned`/`status`. The sample in the report allows `write` on a user's own document, which would let a user make themselves admin.
5. If the same code or keys were ever pushed to another repo (team repo, Classroom, etc.), assume they are exposed there and rotate regardless.
6. Initialise a **fresh** repo from the cleaned files rather than adding a remote to a folder with history.
7. Decide on `firebase.json` / config-file policy (section 3), then rename `com.example.morshedy` and the `morshedy` package name if you want a clean portfolio presentation (optional; needs Firebase re-registration).
8. Clone-test: in a clean folder, follow the README from scratch.
9. Check `git status` / `git ls-files` before the first push for anything in section 3.

Nothing has been pushed or published.

---

## 5. Needs your confirmation

1. **Which code is final?** The zip appears older than the report (table in section 1). Do you have a newer branch? The README currently describes only what the zip contains.
2. **Your contributions.** You referred to your CV, but it was not attached, and the report assigns no individual roles (all seven members are "Development Team"). The README lists your three items as you described them, with a hidden HTML comment to confirm. Suggested accurate wording: *"Firestore data modelling (NoSQL schema design), Flutter UI screens, and user-flow/UX design"* — "database design" alone could be read as relational design. Please also confirm which screens you personally built; I did not claim any specific files.
3. **Team names.** I listed all seven names from the report cover. Do all members agree? Delete the list if not. The PDF's metadata lists one author, but I did not assume that is you.
4. **Screenshots:** are you allowed to reuse images from the university-submitted report? Also, a team member's real name appears in the original captures (I blurred it).
5. **License and ownership.** The report's declaration says the work is the property of Taif University. I left the repo unlicensed. Ask your supervisor/university before choosing a license.
6. **Repository details:** GitHub username/repo name (README uses `<your-username>/yusr`), and whether you want the team members linked to GitHub profiles.
7. **"Journey notes."** Your brief and the report list it as a feature; the code has an itinerary checklist instead. I documented the code.
8. **Flutter version.** I wrote "Dart ≥ 3.10.4, i.e. Flutter 3.38+" from knowledge of Flutter release pairings and could not confirm it with a toolchain here — run `flutter --version` on your machine and adjust.
9. **iOS Google Sign-In.** `Info.plist` has no Google URL scheme; I could not tell whether iOS sign-in worked in the team's build.
10. **Report-only claims** (performance numbers, UAT rates) are quoted as "the report states"; I could not reproduce them.
