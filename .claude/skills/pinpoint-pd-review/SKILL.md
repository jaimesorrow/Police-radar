---
name: pinpoint-pd-review
description: Reviews changes in this repo (Police-radar, a crowdsourced "pin a spotted police vehicle on a map" app) against its actual pre-scaffold state and its domain-specific correctness/safety needs — pin staleness, reporter anonymity, coordinate validation, abuse resistance, and this repo's undefined relationship to the sibling Police-radar-android client. Use this instead of a generic code review, and instead of the sibling repo's own police-radar-review skill (which covers Police-radar-android specifically), for any change here — especially the first commits that add real source code, a chosen platform/stack, or a data model for police-sighting reports.
---

# Police-radar review

## Repo reality check (read this before reviewing anything else)

As of this writing, `Police-radar` contains only `README.md`, `LICENSE`, and a `.gitignore` —
**no source code, no build files, no chosen platform, no CLAUDE.md**. The README is one line:
"An app where people have the ability to pin point on a map that a police vehicle has been
spotted." There is a separate sibling repo, `Police-radar-android` (also owned by the same
account), which is **currently in the identical stub state** — same one-line README, no source
either. Do not assume either repo has "the real app" already; do not assume this repo is a backend
or web client just because a same-named Android repo exists — as of now neither repo has picked a
role.

The existing `.gitignore` here is already Android/Gradle-flavored (`.gradle/`, `build/`,
`local.properties`, `*.iml`, `.idea/`, `*.aab`/`*.apk`, `*.jks`/`*.keystore`,
`google-services.json`). That is either a strong signal this repo is *also* meant to be (or start
as) an Android client, or it's a copy-paste leftover from the sibling repo that hasn't been
reconciled yet. Because this repo has no code yet, most of what a review here can check is about
the **first real commits that scaffold something** — treat this as a pre-flight checklist for that
moment, not a diff-vs-established-invariants review the way a mature codebase gets.

## Checklist

1. **Role vs. `Police-radar-android` is stated, not assumed.** The first commit(s) that add real
   source should say, in the README or a new CLAUDE.md, what this repo actually is (backend/API,
   web client, iOS client, shared spec/design doc, or — if it really is a second native Android
   client — why that duplication is intentional). Flag a PR that silently scaffolds a second
   full native Android app here with no acknowledgment of `Police-radar-android`, and flag a PR
   that picks a non-Android stack (web/backend/iOS) but leaves the Android/Gradle-shaped
   `.gitignore` untouched and unexplained.

2. **Submitted pin coordinates are validated before being stored or shown.** The core write path
   is "user reports a police vehicle at a location." Any endpoint/handler/view-model that accepts a
   lat/lng (or device location fix) must reject out-of-range, null, (0,0)-sentinel, or otherwise
   implausible coordinates before persisting or broadcasting them — an unvalidated pin is a
   correctness bug for a map-pin app, not a nitpick.

3. **Every pin has a timestamp and an enforced staleness/expiry mechanism.** Police vehicles move;
   a pin with no age limit is actively misleading, not just stale data. Any schema, API response,
   or UI list for pins must carry a creation/observed timestamp, and there must be a real
   mechanism — TTL in the query, a scheduled job, a client-side auto-hide, something — that removes
   or visually demotes pins past some age. A design that stores pins with no decay path should be
   flagged even if "we'll add expiry later" — for this domain, staleness handling is part of the
   feature, not a follow-up.

4. **Reporter identity is not exposed by the pin-sharing path.** Reporting a police vehicle's
   location is the kind of report some users would want to make anonymously (fear of retaliation,
   surveillance, harassment). Check that whatever submits a pin does not attach a persistent user
   ID, device ID, or precise reporter-location trail that other users (or an admin/debug view) can
   read back. If accounts get introduced later, that's a deliberate product decision worth calling
   out explicitly in review — same as it would be in any app whose core action is reporting on law
   enforcement.

5. **The submit-a-pin path has some abuse/spam resistance.** A crowdsourced safety feature with an
   unrated, unlimited-write "report a sighting" endpoint invites false reports and harassment
   (fake pins, flooding one area, targeting a specific street). Look for *some* mitigation —
   rate-limiting, per-device/session throttling, downvote/flag-as-wrong, geographic
   plausibility checks (e.g. a device can't plausibly report two far-apart locations seconds
   apart) — before treating "submit a pin" as feature-complete.

6. **Map/geolocation API keys are never committed.** Whatever mapping SDK gets chosen (Google
   Maps, Mapbox, Apple MapKit, OSM+Leaflet, etc.), confirm its key/config lives outside version
   control and that the actual file it's stored in is covered by `.gitignore` — don't just trust
   that `local.properties`/`google-services.json` are listed already; check that the new code's
   real config file matches what's actually ignored.

7. **No overclaiming legality across jurisdictions.** Community reporting of police locations
   (Waze-style) is legally treated differently from radar-detector *hardware*, and rules on both
   vary by state/country. Flag any copy, marketing text, or in-app disclaimer that asserts the
   feature is legal everywhere, or that frames the app as a tool to evade law enforcement, without
   a basis for that claim — this app has no jurisdiction scoping today, so don't let one get added
   implicitly through a stray string.
