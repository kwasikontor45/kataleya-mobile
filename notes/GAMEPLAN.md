# Kataleya — Gameplan & Handover

*Written 2026-09-08. For Bonesaw, working across this chat (design/strategy) and Claude Code
(implementation). Grounded in the actual repos (`kataleya-main.zip`, `kataleya-mobile-main.zip`),
not the summaries in their own docs — a few things below correct what `ROADMAP.md` and the
READMEs undersell.*

## ✅ Checkpoint — pairing mechanism finished, closed 2026-09-19 ~14:35

**STARTING ~13:58 · [claude]** → **DONE ~14:35 · [claude]**

Bonesaw's brief for this one, verbatim intent: simple/clean, very secure, works offline and
online, works face-to-face and at a distance, grandma-friendly, "low-fi surface / high-tech
underneath," and — explicitly — **our own design signature, not mainstream, not somebody else's
style.** That last part ruled out the obvious mainstream answer (a generic QR-code scan-to-pair
flow) on purpose, not by oversight — see the design call below.

**Shipped, all round-trip tested against the real code actually in `index.html` before shipping,
not just a standalone prototype:**

1. **Word-form attunement codes**, replacing the raw 64-character hex blob for the one-time
   sponsor/sponsee handshake specifically. 256 unique words, 1 byte ↔ 1 word (no bit-packing
   ambiguity to get subtly wrong), drawn from Kataleya's own existing vocabulary (light, water,
   garden, river, ember, dawn, dusk, plus plain nature words to round out the count) — then
   filtered so no two words are within one-letter edit distance of each other, so nothing in the
   list can be misheard or mistyped for something else (cove/cave, dock/lock, flow/glow — 43 such
   pairs existed in the first draft and got removed). Verified: 20,000 random-key round trips,
   all-zero and all-max edge cases, case/whitespace tolerance, clean rejection of garbage input —
   all against the actual embedded code, extracted and executed for real, not eyeballed. Still
   accepts an old hex-format code too (verified), so nothing already shared under the old scheme
   breaks. Honest tradeoff, stated in-app, not oversold: 32 words is a real block of text, not a
   4-digit PIN — a full 256-bit key doesn't compress smaller than that without weakening it, and
   this doesn't attempt to. It's genuinely easier to read, type, and compare by eye than 64 hex
   characters, which is the real, honest improvement being claimed.
2. **A verification phrase**, shown once both people are attuned — same pattern as Signal's
   "safety numbers": a short (4-word) phrase derived from both public keys hashed together in a
   canonical sorted order, so it comes out byte-identical on both screens regardless of who
   initiated (verified symmetric across 5,000 random keypairs). Not a replacement for the real
   X25519 exchange — a check on top of it, catching the case where a code got tampered with or
   swapped in transit before the two devices ever noticed. This is a real security addition, not
   decoration — purpose-built for the face-to-face case Bonesaw asked for (read it to each other,
   in person, once).
3. **Native OS share sheet** (`navigator.share()`) wired in for both the attunement code and
   outgoing light/water signal codes, feature-detected once at boot. Replaces "copy this, switch
   apps yourself, paste it" with one tap → pick the app (Messages, WhatsApp, email, AirDrop,
   whatever) → done. This is the piece that actually answers "different geolocation" — remote
   sending stops requiring any manual app-switching at all. Falls back to the existing copy-button
   behavior exactly as before on any browser without share support (mainly desktop) — zero
   regression there. Works fully offline too: sharing to another already-installed app needs no
   network, same as the manual-paste path it's improving on.

**Deliberately not built, and why — this needs to be an honest, stated call, not a silent gap:**
a QR-code scan-to-pair flow is the obvious mainstream answer to "face-to-face pairing," and it's
exactly the one Bonesaw asked to avoid ("not mainstream, not somebody else's style"). But
independent of that, it also wasn't safe to build tonight regardless: a real QR encoder needs
correct Reed-Solomon error correction, finder patterns, and mask-pattern selection, and a real
scanner needs either a vendored decoder library or the native `BarcodeDetector` API — which iOS
Safari doesn't reliably support, the exact platform just audited for gaps two sessions ago. Hand-
writing either from memory, with no way to test against a real camera or a real second device
right now, risks shipping something that looks done but silently fails in the one moment (a
sponsor pairing, possibly during a hard moment) where silent failure matters most. The word+share
+verification-phrase combination above serves the same real goals — fast in-person pairing, works
cross-platform, no new heavy dependency — through a path that could actually be verified correct
before shipping, and it's more genuinely "Kataleya's own style" than a QR flow borrowed from every
other app would have been anyway. If a camera-based flow is still wanted later, that's a real,
scoped, separate piece of work — not something to bolt on unverified.

**Not touched, on purpose:** the underlying crypto (X25519 + XChaCha20-Poly1305), the relay
worker, the manual-paste fallback mechanism itself, and the light/water signal codes (those stay
hex — they're variable-length, sometimes long with a carried water message, and word-encoding
something that could run to 180+ bytes would make transcription worse, not better; share sheet is
the real fix for that side, already shipped above).

`index.html` in hand, current. `GAMEPLAN.md` here is the only doc touched this session.

**Next session starts here:** two honest loose ends from tonight, either is a reasonable pick —
(a) the room's remaining "alive" work (mirror/vault/scars data actually shaping the room itself,
not just the whisper overlay — flagged since Phase 2, still not started), or (b) if a camera-based
pairing flow is wanted after all despite tonight's call against it, that's real scoped work on its
own (vendoring a tested QR library, verifying against real devices) and shouldn't be rushed into a
single session the way tonight's word/share/verify-phrase approach could be.

## ✅ Checkpoint — web app "alive room" pass, closed 2026-09-19 13:52

**STARTING 12:54 · [claude]** → **DONE 13:52 · [claude]**

Everything shipped this session, in one line each — full detail with reasoning is inline below,
under Phase 2/3:

1. Phase 0 reference archive (`index.legacy.html`) — untouched snapshot before any of this.
2. Continuous phase-color blend — the four phases stopped hard-cutting, now drift smoothly.
3. Two always-on animation-loop bugs fixed (burn river, mirror sweep) — real perf win app-wide.
4. Pairing screen redesigned around light/water language — water is now a real second gesture.
5. Color palette corrected to the documented Circadian Canonical Hours system.
6. Room orb pulse depth now drifts with time of day too, not just color.
7. Settings-screen audit — dev/demo tooling moved to the terminal, one mislabel fixed.
8. Phase 1 architecture call resolved: **hybrid — PWA now, native only if a real gap forces it.**
9. iOS Safari pass — audited against real gaps, shipped an install nudge, logged the rest.

`index.html` and `index.legacy.html` in hand, both current. Nothing from tonight is half-shipped.

**Not run this session (sandbox has no git repo / `arc` tool):** `arc clean` and the
`~/dev-notes/hq-main/reference.md` update from the usual closeout protocol don't apply here —
noting that plainly rather than skipping it silently.

**Superseded — see the pairing-mechanism checkpoint above, closed the same day.** The line below
is kept for the historical record of what this checkpoint's report actually said at the time;
don't treat it as the current pointer.

~~Next session starts here: the pairing mechanism (still manual copy-paste under the good
language from #4 above — QR-code or similar is the real next step, not started)~~

## Read this first if you're Claude Code

- `kataleya-mobile`'s `AGENTS.md` says it plainly: **Expo has changed since your training data.**
  Read `https://docs.expo.dev/versions/v57.0.0/` before writing any Expo/RN code in that repo.
  Don't rely on memorized Expo APIs.
- The web app (`kataleya-main`) is **not** the toy the README implies. `index.html` is 3,311
  lines, single-file, no build step, vanilla JS/CSS. It has ~9 real screens (`room`, `bridge`,
  `cover`/cocoon, `terminal`, `pairing`, `burn`, `scars`, `vault`, `settings`, `mirror`), a
  phase-based color system keyed to time of day, canvas animations (ouroboros, EKG), a
  typewriter-driven PIN-gated log, and real E2E crypto (X25519 + XChaCha20-Poly1305, vendored in
  `vendor/kataleya-crypto.bundle.js`, not CDN-loaded — this is deliberate, don't "fix" it by
  switching to a CDN import). Read the whole file before touching it; a lot of behavior lives in
  inline event handlers and a shared `state` object, not in separated modules.
- `kataleya-mobile` is genuinely Phase 0 — `App.js`/`index.js` are the stock Expo blank template.
  No feature code exists yet. Nothing here is a partial port to finish; it's a fresh build against
  the web app as reference.
- Package name is decided and set: `studio.kontor.kataleya` (Android-only scope for now, by
  Bonesaw's call — no iOS bundle ID needed yet).

## Decision — resolved 2026-09-08

> **Kataleya is personal-first.** It's an instrument Bonesaw uses himself. The Play Store /
> generic-audience framing is secondary, not the driver.

This resolves the open question from the first draft of this doc. Consequence for direction: the
current app's voice — the terminal calling itself `the garden's vitals`, the cocoon's hold-to-
return ring, re-entry detection, `engine_room`, hour-scars — is **not** a barrier to smooth away
for strangers. It's the actual point. Reimagining means going deeper into that instrument for an
audience of one, not adding onboarding-for-strangers translation layers. Don't genericize copy or
metaphors in the rebuild without Bonesaw explicitly asking for that.

Practical effect on sequencing: Track B's stranger-facing paperwork (content rating answers, Data
Safety form language, generic store descriptions) stays real but drops in priority relative to
Track A. It's fine for those Play Store items to sit half-finished for a while — nothing about
personal-first blocks eventually shipping, it just means the reimagined experience for Bonesaw
comes first, store polish comes after it's right.

## Track A — Reimagined build (web + mobile), personal-first

Feature set and full visual direction are still to be designed in detail (that's ongoing design
conversation, not a Claude Code task) — but the frame is now set, so here's a real phase plan
Claude Code can start sequencing against as design decisions land, rather than a placeholder.

**Phase 0 — keep, don't discard. ✅ done 2026-09-08.** `index.legacy.html` is a byte-identical
archive of the pre-rebuild `index.html`, made before any edits — nothing in the existing ritual
(ouroboros animation, whisper cycling, phase color system, mirror/vessel EKG rendering) is at
risk. Treat `index.legacy.html` as the reference implementation to draw from going forward, keep
it un-edited.

**Phase 1 — architecture decision, made concrete.** Design brief: *clean, inevitable, just works,
self-healing, useful, alive — an elegant, inevitable living/breathing organism.* Concretely for
architecture, "self-healing" is the load-bearing word here: fewer things that can stay broken, no
dependency chain to snap, no build step to fail, no app-store review gate between a fix and the
phone in Bonesaw's hand. The current web app already has this almost by accident — single file,
no build step, service worker just re-caches on next load. That property is not a limitation to
graduate out of; it **is** the self-healing mechanism. Default position: don't replace the web
core with a native rebuild — layer the minimum native surface on top of it that actually earns its
complexity. Push notifications for the pairing "light arrived" case is the one candidate raised so
far worth that trade (native has a real, mobile-only capability there — background delivery the
current web PWA can't do). Everything else defaults to staying in the self-healing web core unless
a concrete capability gap forces the trade, not because native "feels more like an app."
`ROADMAP.md`'s old non-goal (no shared codebase between web and mobile) stays the default too —
don't reach for React Native Web / Expo Router web target just to avoid writing two
implementations; that abstraction layer is exactly the kind of thing that erodes the self-healing
property for the sake of not repeating yourself.

**Phase 2 — room, reimagined first.** Same design brief applied to the room screen specifically:

- ✅ **Shipped 2026-09-08 — color field, continuous.** `applyPhaseVars()` now blends `--accent`/
  `--shadow`/`--highlight`/`--ambient`/`--rim` continuously across each phase's transition window
  (new `interpolatedPhaseVars()`, hex RGB lerp, ticked every 15s instead of the old 60s discrete
  check) instead of hard-cutting at the boundary — verified it lands exactly on the original hex
  values at each boundary (`t=0.00`/`t=1.00`), so nothing about the existing palette actually
  changed, it just stopped snapping. Deliberately scoped narrow: `state.phase`, `displayName`,
  `existential` copy, and the whisper set still snap discretely (text can't blend) — only color
  changed. A manual `/phase` override still applies its exact static colors, no blending — that's
  a deliberate discrete choice, not something to smooth over. `palette()` itself (still used by
  ~10 other call sites for discrete logic) is untouched.
- ✅ **Shipped 2026-09-19 — orb behavior now drifts with time too, not just color.** `breathMs`
  existed in the phase data since the original palette but was never actually read anywhere —
  confirmed dead before touching it. New `roomLifeAmplitude()` interpolates it continuously (same
  `getPhaseBlend()` used for color) into a 0.85–1.25 multiplier on the room orb's idle pulse
  depth — nyx's slower 4000ms `breathMs` now reads as a calmer, deeper swing; desire's brisker
  2500ms reads as smaller and quicker. Deliberately scoped to depth/amplitude only, never cadence
  — the actual pulse *timing* stays exactly as driven by `state.breathTechnique` (Bonesaw's own
  explicit 4-7-8/box/coherent choice), since that same global loop also drives the guided-practice
  overlay and messing with its timing would break the exercise itself. Scoped to the room orb
  alone (checked by element identity against `$('room-orb')` each frame) — bridge/cover/wave/
  onboarding orbs are byte-for-byte untouched. Recomputed at most once/second inside the 60fps
  loop, not every frame, since the underlying value only needs to move on the same slow cadence as
  the color blend.
- ✅ **Shipped 2026-09-08 — fixed two always-on animation loops causing real sluggishness.**
  Found by reading the code, not guessing: `animateRiver` (mercury river, burn screen) and the
  mirror scanline/heartbeat sweep both did their screen-gated work correctly, but called
  `requestAnimationFrame(self)` *unconditionally*, outside that guard — so both ran forever, every
  frame, from page load, regardless of which screen was actually showing. The mirror loop was the
  worse of the two: `document.querySelectorAll('#vessel-svg .organ-label')` +
  `getBoundingClientRect()` (layout-forcing) on every single frame, forever, even on totally
  unrelated screens like room or vault. Fixed: both loops now only keep rescheduling themselves
  while their own screen is active, and re-arm (`armMirrorSweep()`, and a direct call in
  `enterHooks.burn`) on the way back in. This alone should measurably improve general
  responsiveness across the app, not just on those two screens — main-thread contention from an
  always-on loop slows everything sharing that thread. `breathLoop` and `waveRideFrame` (orb
  breathing) were audited too and are correctly gated already — left untouched.
- *Alive* / *inevitable*, continued — the room's state should keep moving while the app is closed,
  not only compute fresh the instant it's opened. Right now growth stages, mood trends, and the
  whisper cycle are all rendered live off stored data; "alive" means the room reflects what
  happened while Bonesaw was away without being asked — an organism that grew or wilted unwatched,
  not one that only changes when tended. Not started.
- ✅ **Shipped 2026-09-19 — settings-screen audit, first pass.** Went section by section against
  the brief. Kept as genuinely necessary: name/sobriety date (can't be inferred), breath technique
  (real agency, not friction — also practice-critical, left untouched), clinician PIN section
  (explicitly exempted — PIN/security-critical), backup & restore (the actual safety net for
  local-only storage, more important than ever given the iOS storage-eviction note above — read
  it fully before deciding not to touch it, left completely alone). Removed: "preview with 30 days
  of sample history" and "clear seeded preview entries" — both dev/QA tooling that had no business
  sitting next to a real danger control on the app's one actually-dangerous screen. Nothing
  deleted: `/seed` already existed as a terminal command calling the exact same function the
  button did, so the button was a true duplicate — removed outright. `/clear-demo` had no terminal
  equivalent, so one was added (same `clearDemoEntries()` function, same confirm-dialog wording,
  byte-for-byte) before its button was removed — capability preserved, just relocated to the
  screen the app's own convention already uses for this register (`/reset`, `/phase` live there
  too). Caught and fixed one real regression risk while doing this: the settings button's `/seed`
  guarded against silently overwriting real journal/mood/sobriety data with a `confirm()` first;
  the terminal's bare `/seed` didn't have that guard — ported it over so nothing got less safe in
  the move. Also fixed one honest mislabel while in this section: "haptics (simulated)" → just
  "haptics" — `haptic()` genuinely calls `navigator.vibrate()`, it was never simulated, the label
  was just wrong.
- *Useful* is the guardrail against the metaphor eating the function — the real test for every
  change in this phase: does it make a bad moment (2am, mid-craving) easier, or does it just make
  a good moment prettier. Beauty that doesn't clear that bar gets deprioritized.
- Room should draw on the mirror/vault/scars data that already exists rather than that logic
  living only in the separate butterfly-overlay bolt-on — a rough week should show up as weight in
  the room itself, not just in a dedicated overlay screen.

**Phase 3 — the ritual screens.** Bridge, cocoon, terminal, mirror — the screens that give the
app its distinct voice. Carry forward specific mechanics worth keeping deliberately (re-entry
detection, hold-to-return, the PIN-gated typewriter log) rather than rebuilding from a blank
description of what they do.

*Note added 2026-09-08 — pairing/sponsor-sponsee flagged as too complex for a personal-first
instrument, ✅ first redesign pass shipped same session.* Bonesaw's own words: "my grandma will
freak out if she's my sponsor." Root cause found by reading the code: the app's actual religion —
Bonesaw's words — is attunement expressed as light, color, and water; the pairing screen was the
one place that language broke down into raw technical chrome (`[ sponsor_signal ]`, "connect",
"disconnect", bare code-paste fields) around a single poetic phrase ("send a light") that had
nowhere near enough company. Shipped: header renamed to `[ light_and_water ]`; "connect"/
"disconnect" renamed to "attune"/"release" throughout; **water added as a real second gesture**
alongside light, not just copy — `pcEncryptSignal()` already accepted an arbitrary message
(discovered mid-fix, not a new mechanism), so "send water" now optionally carries a short
message (140 char cap) the same way light carries none, encrypted identically. Arrival display
(manual decrypt panel, relay history list, and the arrival toast) is now driven by what was
actually sent (🕯️ light vs 💧 water, with the words if any) via a new shared `pcDescribeSignal()`
helper, instead of the old hardcoded "a light arrived." string that would've lied the moment
water existed. The manual copy/paste mechanism underneath is untouched on purpose — same
zero-connection-required fallback, same real crypto (X25519 + XChaCha20-Poly1305) — this pass
only ever touched the words around it and added the one new message type. All new element ids
cross-checked against their JS references; none dangling. QR-code or fully automatic delivery
(so the person never sees a code at all) is still open, noted as the next real step, not done —
tonight's pass got the language right, not the mechanism.

*Note added 2026-09-08 — color palette discrepancy found, ✅ fixed 2026-09-19.* Bonesaw's own
documented "Circadian Canonical Hours" system (`/topics/design-system.md`): choice `#5ec8ed`,
desire `#f6c177`, still-pine `#c4a7e7`, nyx `#ea9a97`. The `PHASES` object in `index.html` (whose
own comment claimed "verbatim from constants/palettes.ts") had drifted on three of four accents:
choice was `#c8f563`, still-pine was `#9ccfd8`, nyx was `#eb6f92` — only desire matched. Confirmed
with Bonesaw which side was authoritative (the documented system) before touching anything, per
the note below from when this was first found. Fixed: accents for choice/still-pine/nyx now set
to the exact documented hex values; desire's accent (already correct) and all of desire's
shadow/highlight/ambient/rim were left completely untouched, byte-for-byte. For the three
corrected phases, `shadow`/`highlight`/`ambient`/`rim` were regenerated rather than left
mismatched to the new accent's hue — derived via HSL from each new accent's own hue, using the
same hue-stays-with-accent, banded-saturation/lightness pattern the app's four tokens already
consistently followed (verified numerically across all four original tokens before deriving the
replacements, so this isn't an invented convention). Re-verified the continuous phase-blend
interpolation from the earlier "alive room" pass still lands cleanly on every boundary with the
new colors — no regression introduced. `index.legacy.html` (Phase 0's pre-rebuild reference) is
untouched, as always — this fix only ever lives in the working `index.html`.

*Phase 1 architecture call — ✅ resolved 2026-09-19.* Bonesaw's decision, asked directly rather
than assumed: **hybrid — installable PWA is the real answer now; native stays deferred, not
decided against, and only gets built if a specific concrete capability gap actually forces it**
(the standing candidate is real OS-level background push for the pairing feature, not yet needed).
This closes the loop the 2026-09-08 note above left open — Bonesaw had pushed back on treating PWA
as a foregone conclusion, so it got asked outright instead of re-assumed. `kataleya-mobile`'s own
native-rebuild phases (2–6 below) stay fully blocked until/unless that capability gap shows up;
until then, all mobile-facing work happens in `kataleya-main`'s `index.html`.

*iOS Safari pass — ✅ shipped 2026-09-19*, prompted by Bonesaw flagging "Safari can be weird" right
as the hybrid-PWA call landed. Audited `index.html`'s `<head>`, `manifest.json`, and `sw.js`
against known iOS Safari PWA gaps rather than guessing:
- **Already solid, no fix needed:** `apple-touch-icon`, `apple-mobile-web-app-capable`,
  `apple-mobile-web-app-status-bar-style`, `apple-mobile-web-app-title`, and
  `viewport-fit=cover` were all already in place, and the CSS already uses
  `env(safe-area-inset-*)` throughout — the notch/home-indicator safe-area work was already done.
  `navigator.vibrate` (haptics) is already guarded with an existence check, so it's a silent no-op
  on iOS (which has no vibration API) rather than a crash — confirmed, left alone.
- **Real gap, fixed:** iOS has no `beforeinstallprompt` — Safari gives no native "install this"
  affordance, so nothing ever told an iOS visitor that Add to Home Screen was an option, or why it
  matters. That matters more than cosmetics here: a tab that's never added to the home screen also
  gets weaker on-device storage guarantees under iOS's storage eviction policy, and this app's
  vault/journal live entirely in that storage with nothing server-side backing it up. Shipped a
  small dismissible top banner (`#ios-install-nudge`), shown once (persisted via the existing
  `store` helper) to iOS Safari visitors not already running standalone, pointing them to Share →
  Add to Home Screen. The existing backup-code export/import feature is the real safety net
  regardless of storage eviction — this nudge is about reducing how often that net gets needed,
  not a replacement for it.
- **Real gap, logged not fixed:** no `apple-touch-startup-image` (splash screens) — iOS shows a
  blank flash on launch without one. Cosmetic, low priority, not done tonight.
- **Documentation-vs-code gap, logged not fixed:** `/areas/kataleya.md`'s own notes describe the
  pairing feature as using "Web Push," but the actual code only polls the relay
  (`pcRelayCheck()`, every 20s, only while the pairing screen is open) — there's no `push` event
  listener anywhere in `sw.js`, so there's no real Push API dependency to break on iOS in the
  first place. Worth a quiet correction to that note at some point; not urgent, not user-facing,
  and not something this pass touched.

**Phase 4 — vault + scars/burn.** Journal, mood-trend visualization, growth stages tied to
`daysSince()`. Local-first storage decision for mobile (SQLite via `expo-sqlite` was the prior
candidate — revisit once Phase 1's architecture call is made, since it may change).

**Phase 5 — pairing.** E2E sponsor/sponsee signal, ported last as before — most backend-dependent
piece, least central to a personal-first instrument (still real and worth keeping, just not
first).

**Phase 6 — polish + store packaging.** Only once Phases 1–5 feel right for actual daily use —
this is where Track B's remaining paperwork (descriptions, content rating, screenshots) picks
back up.

**Next action for this track:** continue the design conversation (visual direction, what changes
vs. what's carried forward from the reference implementation) before Claude Code starts Phase 1.

## Track B — Store readiness (unblocked, safe to parallelize)

From `PLAY-STORE-SHIPPING.md`, still accurate as of 2026-09-07:

- [x] Package name decided — `studio.kontor.kataleya`
- [x] Android-only scope confirmed
- [ ] `npm install -g eas-cli`, then `eas login` — neither done
- [ ] Publish `privacy-policy.md` at a real public URL — blocked on `kontor-studio`'s deploy
      pipeline (`arc deploy`'s Cloudflare token is dead as of 2026-08-22) or an alternate host
- [ ] Fill remaining bracketed placeholders in `privacy-policy.md` (relay retention window, final
      Android permissions list, real contact method) — some can't be finalized until Track A
      code exists; retention window and contact method could be answered now
- [ ] Draft short (≤80 char) + full (≤4000 char) Play Store descriptions
- [ ] Verify existing icon/adaptive-icon assets meet current Play spec
- [ ] `eas.json` build profiles — scaffolded but unverified against current EAS CLI

**Caution:** several of these (store description copy, "what this app does" framing, content
rating answers) are downstream of the open decision above — a personal-instrument reimagining may
describe itself very differently to Play Store reviewers than the current product-framed copy
does. Fine to do the purely mechanical items (EAS login, icon spec check, deploy pipeline fix)
without waiting; hold off on copy/description drafting until direction is clearer.

## Suggested split while working across both of us

- **This chat (Claude):** design direction, feature/scope decisions, copy and voice work, privacy
  policy language, anything that needs Bonesaw's judgment surfaced and discussed before code gets
  written.
- **Claude Code:** implementation once direction is set — Expo/RN code (checking v57 docs per
  `AGENTS.md`), the `kontor-studio` deploy pipeline fix, EAS setup, and any Track B mechanical
  work that doesn't require a copy/scope decision first.

## Immediate next actions (no blockers, start now)

1. Archive/tag the current `kataleya-main` `index.html` as a reference build before any rebuild
   touches it (Phase 0 above) — Claude Code, do this first, mechanical.
2. Fix `kontor-studio`'s dead `arc deploy` Cloudflare token, or stand up an alternate host for the
   privacy policy — Claude Code, mechanical, no design input needed, can run in parallel.
3. `npm install -g eas-cli` + `eas login` on the dev machine — Claude Code.
4. Verify icon/adaptive-icon assets against current Play spec — Claude Code.
5. Continue the design conversation in chat — visual direction and what's carried forward vs.
   reimagined in Phase 2's room screen — before Claude Code starts Phase 1's architecture work.

## 2026-10-01 · Kataleya wave accessibility + mobile performance pass (deployed)

Worked in `kontor-studio/kataleya-demo`, the live web/PWA source used by the Android TWA. The Expo
`kataleya-mobile` project remains an empty scaffold; no native app or Android wrapper changes were
needed for this pass.

- Added a persisted wave-sound switch in the exercise and Settings. Sound defaults off, uses short
  locally generated soft tones at the start/crest/passing moments, and can be silenced immediately.
  No audio files, third-party libraries, requests, permissions, or user data were added.
- Kept the orb's rise/crest/recede arc; added a very slow 18-degree drift and a softer opacity halo.
  Reduced-motion preference now keeps the wave orb at a steady size and removes the halo transition.
- The shared breathing loop now updates only visible-screen orbs/canvases, drops disconnected
  entries, and sleeps while the document is hidden. Wave copy is written only when its phase changes.
- Bumped the service-worker shell cache so an updated install also gets the new behavior offline.
- `node --check kataleya-demo/app.js`, `node --check kataleya-demo/sw.js`, and `git diff --check`
  passed. Commit `07ebfc4` was pushed to both configured remotes and deployed via
  `arc deploy kontor-studio`. Live checks returned HTTP 200 for the app shell and confirmed the
  new `waveSound`/mute/foreground-loop code and `kataleya-shell-v9` service worker. Physical-device
  interaction and performance QA were unavailable; reduced background work is an expected
  improvement, not a measured result.

## Next project milestone · Kataleya in the existing Waydroid environment

Waydroid was already installed, initialized, and privacy-sanitized on 2026-08-23 (see the shared
SOT and security reference); the next task is not a fresh install. Use it as the Android test target
while the phone is unavailable. Check the current Kataleya beta TWA or installed PWA at a phone
viewport, including startup, wave interaction, sound default/off behavior, reduced motion, and
touch responsiveness. Keep the no-Google-account setup intact; sideload an existing beta APK if
needed. Use the results to decide whether the PWA/TWA needs tuning before considering native work.
