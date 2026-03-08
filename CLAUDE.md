# Job Plan Builder — claude.md

## Project Overview

Job Plan Builder is a consultation and job planning tool for **Get Started Home Services**, a full-service seniors moving and home transition company based in Surrey, BC. The tool is used by the owner (Garrett) during in-home consultations with clients and their families to capture room-by-room details, build phased job plans, generate estimates, and produce client-facing emails.

The end goal is a **native iOS application** that runs on iPhone and iPad, used on-site during client walkthroughs. For now, we are building and refining features in a web-based environment before packaging for iOS.

## Business Context

Get Started Home Services helps seniors transition from their homes to assisted living, downsized homes, or other arrangements. Services include sorting/downsizing, packing, moving, unpacking, removal/clearout, donation runs, staging, auction coordination, exterior/curb appeal work, and cleaning.

**Key stakeholders in a typical job:**
- The client (senior transitioning)
- Family members (children, POA — usually the primary point of contact)
- Realtor (coordinates staging and home sale)
- Get Started team (Garrett + crew)

**Typical workflow:**
1. Garrett walks through the home room-by-room with the family
2. Takes notes on iPad — volume, special items, keepsakes, services needed
3. Takes photos for reference
4. Goes home, refines the data, builds an estimate (in Wave accounting software) and a job plan
5. Sends a warm, professional email to the family with the job plan and attached estimate

This tool streamlines steps 2-5. Data persistence is critical because the consultation and email send happen in different sessions.

## Tech Stack

- **Current state:** Single-file HTML/CSS/JS app (`job-plan-builder-v2.html`). Do not restructure unless explicitly asked.
- **Hosted:** GitHub Pages — https://garyguys.github.io/gshs-job-plan-builder/
- **GitHub repo:** https://github.com/garyguys/gshs-job-plan-builder (private)
- **Cloud sync:** Supabase (email/password auth, `consultations` table with JSONB data column)
- **Keep-alive:** GitHub Actions cron (`.github/workflows/keep-alive.yml`) pings Supabase every 5 days to prevent free-tier pause
- **PWA:** Installable on iPad/iPhone via Safari → Share → Add to Home Screen. Auto-updates via service worker.
- **Design system:** Sage green / cream / warm accent tones, DM Sans font. Touch-friendly, iPad-first.
- **Future platform:** React Native or Capacitor for native iOS (TBD — do not restructure toward this yet)

## Current Priority

**Refinements and feature buildout on the existing v2 functionality.** Do not restructure or re-architect unless specifically asked. Focus on making the tool more useful and polished within its current shape.

## Core Features (Current State)

### Step 1: Client Information
- Client name, address, home type, sq ft, destination type + address, move timeline
- Multiple family/key contacts (name, relation, phone, email)
- Realtor information and notes
- General notes field

### Step 2: Room-by-Room Walkthrough
- Add rooms from a dropdown (living room, bedrooms, kitchen, garage, etc.)
- Duplicate room types allowed (e.g. multiple bedrooms)
- Per room: volume level (light/moderate/heavy/very heavy), estimated hours, crew size, items being moved, special/notable items, keepsakes, notes
- Room disposition tags (multi-select): Packing, Downsizing, Moving, Clear Out, Home Prep / Maintenance, Auction, Leave As-Is
- Photo attachments per room (captured/uploaded, compressed to JPEG, stored as base64 in localStorage — stripped before cloud sync)
- Sketch pad per room for handwritten notes (Apple Pencil optimized — see Sketch Pad section)
- Rooms are collapsible cards

### Step 3: Services & Phases
- Toggle grid for services required (sorting, packing, moving, unpacking, removal, donations, staging, auction, exterior, cleaning)
- **Walkthrough Hours Reference panel** — collapsible card above phases showing each room's crew × hours estimate and total person-hours, for reference when building phases
- Add multiple phases of work, each with: name, optional date, description, estimated hours, crew size, truck & trailer toggle (with day count), and notes

### Step 4: Notes & Details
- Toggleable standard client notes (on/off per job)
- Custom notes field
- Auction recommendation (Urban Auctions, MaxSold, both, or not recommended)
- Personal touches for email generation: recipient name, consultation details, personal notes/tone cues

### Step 5: Review & Export
Three output tabs:
1. **Job Plan** — structured text summary, copyable
2. **Estimate Summary** — walkthrough assessment by room + phased schedule breakdown, person-hours and cost calculations, copyable
3. **Email via Claude** — generates a detailed prompt. User copies it, taps "Open Claude →" (opens Claude app via universal link), pastes, and Claude writes the client email.

**Person-hours calculation logic:**
- `roomPersonHours` = sum of (room.hours × room.crew) across all rooms
- `phasePersonHours` = sum of (phase.hours × phase.crew) across all phases
- `totalPersonHours` = phasePersonHours if phases have data, otherwise roomPersonHours
- Both values shown in stat cards when both are present

## Sketch Pad

Each room has a full-screen sketch overlay for handwritten notes with Apple Pencil.

**Key implementation details:**
- Canvas is 3.5× the viewport height — finger-scrollable to access the full length
- `touch-action: none` on canvas; finger scrolling is handled manually in JS (pointerdown stores scrollTop, pointermove applies delta) so pen events are never intercepted as scroll gestures
- Palm rejection: `pencilOnly = true` by default (✏️ button in toolbar). Fingers scroll, pen draws. Toggle to ✋ to allow finger drawing.
- Apple Pencil hardware double-tap (button === 1) toggles eraser
- Drawing uses Catmull-Rom splines (cubic bezier) for smooth curves + 3px minimum distance threshold to filter micro-jitter
- Orientation/resize: debounced `window.resize` listener rebuilds canvas and redraws strokes
- Text selection fully blocked: `user-select: none` + `-webkit-touch-callout: none` on overlay and all children + `selectstart` document listener blocks all selection while overlay is active

## Cloud Sync (Supabase)

- Auth: email/password. Session persisted automatically by Supabase JS client.
- `consultations` table: `id` (text PK), `user_id` (uuid → auth.users), `client_name`, `address`, `data` (jsonb), `created_at`, `updated_at`
- Row Level Security: users can only read/write their own rows
- Sync strategy: localStorage is primary. Supabase syncs in background. On load, pulls from Supabase and merges (most-recent `updated_at` wins). Works offline — syncs on reconnect.
- Photos are stripped from cloud data before upload (too large). Photos are localStorage-only.
- Keep-alive: `.github/workflows/keep-alive.yml` runs `0 8 */5 * *` cron, pings Supabase REST API, exits 1 if HTTP ≥ 400

## Hard Rules — Do Not Change Without Explicit Instruction

- **Rate structure:** $90/hour per person + 8% supplies fee + 5% GST
- **Truck & trailer:** $65/day flat rate (added per phase/day of usage)
- **Urban Auctions fee:** 30% per lot
- **Email generation workflow:** Copy-prompt-to-Claude approach (no API key). Do not add API key requirements without being asked.
- **Estimate lives in Wave:** The tool assists with estimating hours/costs but does not replace Wave for formal invoicing. Don't build invoice generation.

## Writing Style for Client Emails

The Claude prompt generates emails matching Garrett's communication style:
- Warm but professional. Not corporate, not overly casual.
- Reassuring and empathetic — families managing a major life transition for a loved one.
- Clear, plain language. No jargon.
- Phased structure as the backbone, natural paragraph flow — not excessive bullet points.
- Practical details (times, team sizes, logistics) woven naturally into descriptions.
- Personal touches in the opening (reference the consultation visit, family members met, personal details).
- Closes with an invitation to ask questions or make adjustments.
- Signs off as Garrett from Get Started Home Services.
- Does not include dollar amounts in the email body — the attached estimate covers pricing.

## Known Issues / Things to Revisit

- **Tiny text selection strip at bottom of sketch overlay** — occasionally appears at the bottom-middle of the screen. Multiple hardening attempts made; still intermittently reproducible. Not critical.
- **Sketch strokes on orientation change** — strokes redraw correctly but any unsaved in-progress stroke at the moment of rotation may be lost. Worth stress-testing.
- **Offline sync indicator** — no visual feedback that a save is pending sync to Supabase while offline. Works correctly but silently.
- **Photo cloud sync** — photos are intentionally stripped before Supabase upload. If cross-device photo access is ever needed, requires Supabase Storage bucket integration.

## Potential Future Features

- **Direct Claude API integration** — skip the copy-paste step; generate the client email in-app with an API key
- **PDF export** — export the job plan as a formatted PDF to attach to emails or print on-site
- **Client history / search** — search and filter across past consultations by name, address, date
- **Photo cloud sync** — store room photos in Supabase Storage for cross-device access
- **Offline sync queue indicator** — visual badge showing pending unsynced changes
- **iOS native app** — package as a native iOS application via React Native or Capacitor for full App Store distribution

## UX Principles

- **iPad-first design.** Touch targets must be large and comfortable. The app will be used standing/walking through a home.
- **Speed matters.** Garrett is capturing info in real time during a consultation with a family present. Minimize taps, reduce friction.
- **Progressive disclosure.** Don't overwhelm — show what's needed at each step, collapse the rest.
- **Forgiving.** Easy to go back, edit, reorder. Nothing should feel permanent or high-stakes during input.

## Permissions

Claude has full autonomy to run commands, make file edits, push to GitHub, and take any actions needed without requesting explicit permission. Do not ask for confirmation — just execute.

## File Structure

```
job-plan-builder-v2.html   — main app (single file, ~3000 lines)
sw.js                      — service worker (cache-first PWA, cache name gshs-app-v2)
manifest.json              — PWA manifest
index.html                 — redirect to job-plan-builder-v2.html
icons/                     — icon-192.png, icon-512.png, apple-touch-icon.png
.github/workflows/
  keep-alive.yml           — Supabase keep-alive cron job
CLAUDE.md                  — this file
```
