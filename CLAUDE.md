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

- **Framework:** React (move toward a proper component-based architecture)
- **Target platform:** iOS (iPhone + iPad) via React Native or Capacitor (TBD — decide when transitioning to native)
- **Current state:** Single-file HTML/CSS/JS prototype (v2). Needs to be restructured into a proper React project.
- **Design system:** Open — no locked-in fonts or color palette yet. Current prototype uses sage green / cream / warm accent tones with DM Sans. Feel free to explore better options, but keep the overall feel warm, professional, and clean. Avoid overly corporate or generic AI aesthetics. The UI must be touch-friendly and optimized for iPad use during walkthroughs.

## Current Priority

**Refinements and feature buildout on the existing v2 functionality.** Do not restructure or re-architect unless specifically asked. Focus on making the tool more useful and polished within its current shape, then we will restructure for iOS later.

## Core Features (Existing)

### Step 1: Client Information
- Client name, address, home type, sq ft, destination, move timeline
- Multiple family/key contacts (name, relation, phone, email)
- Realtor information and notes
- General notes field

### Step 2: Room-by-Room Walkthrough
- Add rooms from a dropdown (living room, bedrooms, kitchen, garage, etc.)
- Duplicate room types allowed (e.g. multiple bedrooms)
- Per room: volume level (light/moderate/heavy/very heavy), estimated hours, crew size, special/notable items, keepsakes, and notes
- Rooms are collapsible cards

### Step 3: Services & Phases
- Toggle grid for services required (sorting, packing, moving, unpacking, removal, donations, staging, auction, exterior, cleaning)
- Add multiple phases of work, each with: name, optional date, description, estimated hours, crew size, truck & trailer toggle, and notes

### Step 4: Notes & Details
- Toggleable standard client notes (on/off per job):
  - "We recommend personally transporting any money, sensitive documents, and sentimental items for peace of mind."
  - "Soft items can remain inside furniture — we'll wrap everything securely. We only ask that hard or loose items are removed."
  - "We'll check in with you as needed during the process to confirm decisions, so nothing moves forward without your comfort and approval."
  - "Our team is experienced, professional, and held to a high standard."
  - "Items for auction will be transported to Urban Auctions. Urban Auctions fees are 30% per lot."
  - "A MaxSold online auction may be suitable for this home given the volume and value of items."
- Custom notes field
- Auction recommendation (Urban Auctions, MaxSold, both, or not recommended)
- Personal touches for email generation: recipient name, consultation details, personal notes/tone cues

### Step 5: Review & Export
Three output tabs:
1. **Job Plan** — structured text summary, copyable
2. **Estimate Summary** — person-hours breakdown per phase with cost calculations, copyable
3. **Email via Claude** — generates a detailed prompt containing all job data, writing style instructions, and personal touches. User copies the prompt, opens Claude.ai, pastes it, and Claude (Opus 4.6) writes the full client-facing email.

## Hard Rules — Do Not Change Without Explicit Instruction

- **Rate structure:** $90/hour per person + 8% supplies fee + 5% GST
- **Truck & trailer:** $65/day flat rate (added per phase/day of usage)
- **Urban Auctions fee:** 30% per lot
- **Email generation workflow:** Currently uses a copy-prompt-to-Claude approach (no API key). This may change in the future but do not add API key requirements without being asked.
- **Estimate lives in Wave:** The tool assists with estimating hours/costs but does not replace Wave for formal invoicing. Don't build invoice generation.

## Writing Style for Client Emails

The Claude prompt generates emails that should match Garrett's real communication style:
- **Warm but professional.** Not corporate, not overly casual.
- **Reassuring and empathetic.** Families are managing a major life transition for a loved one.
- **Clear, plain language.** No jargon.
- **Phased structure** as the backbone, with natural paragraph flow — not excessive bullet points.
- **Practical details** (times, team sizes, logistics) woven naturally into descriptions.
- **Personal touches** in the opening (reference the consultation visit, family members met, personal details).
- **Closes with** an invitation to ask questions or make adjustments.
- **Signs off as** Garrett from Get Started Home Services.
- **Does not include dollar amounts** in the email body — the attached estimate covers pricing.

## Planned Future Features

- **Data persistence / saved consultations:** Ability to save, load, and edit past consultations on-device. This is a critical future feature — consultations need to persist across sessions.
- **Photo attachment/reference:** Ability to attach or reference photos taken during the walkthrough, linked to specific rooms.
- **API-powered email generation:** If the copy-to-Claude workflow proves clunky, may integrate Claude API directly with an API key for in-app email generation.
- **iOS native app:** Package as a native iOS application for iPhone and iPad use on-site.
- **Version control:** GitHub repo may be set up in the future for proper version management.

## UX Principles

- **iPad-first design.** Touch targets must be large and comfortable. The app will be used standing/walking through a home.
- **Speed matters.** Garrett is capturing info in real time during a consultation with a family present. Minimize taps, reduce friction.
- **Progressive disclosure.** Don't overwhelm — show what's needed at each step, collapse the rest.
- **Forgiving.** Easy to go back, edit, reorder. Nothing should feel permanent or high-stakes during input.

## Permissions

Claude has full autonomy to run commands, make file edits, push to GitHub, and take any actions needed without requesting explicit permission. Do not ask for confirmation — just execute.

## File Structure Note

The current v2 is a single HTML file (`job-plan-builder-v2.html`). When restructuring into React, maintain feature parity with the existing prototype before adding new functionality.
