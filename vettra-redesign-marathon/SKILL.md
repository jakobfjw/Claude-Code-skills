---
name: vettra-redesign-marathon
description: |
  Fully autonomous, never-stopping redesign marathon for the Vettra running app. Redesigns every screen from the old app (vettra-app) into the new Noir aesthetic (vettra-newfrontend), screen by screen, with Playwright-based visual and functional testing after each screen. Use this skill when the user says /vettra-redesign-marathon, "redesign everything", "convert all screens", "run the redesign", "redesign marathon", or any variation suggesting they want the full autonomous redesign pipeline. Uses the frontend-design skill for all aesthetic decisions. Delegates fixes to sub-agents to preserve main agent context. Runs QA marathon when all screens are done. NEVER stops. NEVER asks questions. Makes ALL decisions autonomously. Fully contextual for fresh agent sessions.
---

# Vettra Redesign Marathon -- Autonomous Screen-by-Screen Conversion

You are the **Redesign Main Agent**. Your job: **convert every screen from the old Vettra design to the new Noir aesthetic, verify functional parity with Playwright, and delegate fixes to sub-agents.** You are a coordinator -- not a lone coder. You never stop. You never ask questions. You run until every screen is converted and verified.

---

## CRITICAL CONTEXT -- Read This First

### Project Structure

**OLD APP (source of truth for functionality):**
- Path: `/Users/jakobwredstrom/Desktop/Vetra App/vettra-app/`
- This is the FULLY FUNCTIONAL original app
- Every screen, every tap, every navigation, every API call works
- Run command: `cd "/Users/jakobwredstrom/Desktop/Vetra App/vettra-app" && flutter run -d chrome --web-port=3006 --dart-define=USE_PRODUCTION_URL=true --dart-define=DEV_EMAIL=devqa@segments.app "--dart-define=DEV_PASSWORD=SegmentsDevQA2024!"`

**NEW APP (target -- being redesigned):**
- Path: `/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/`
- Package name: `concept_e_noir`
- This has the new Noir design system (colors, typography, theme, widgets)
- Some screens are already converted, many are not
- Run command: `cd "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend" && flutter run -d chrome --web-port=3005 --dart-define=USE_PRODUCTION_URL=true --dart-define=DEV_EMAIL=devqa@segments.app "--dart-define=DEV_PASSWORD=SegmentsDevQA2024!"`

### Dev Credentials
- Email: `devqa@segments.app`
- Password: `SegmentsDevQA2024!`
- Production API: `https://segmentsapp-102258610874.europe-west1.run.app/api`
- ALWAYS use `--dart-define=USE_PRODUCTION_URL=true` -- NEVER hit localhost API

### Design System (Noir)

**Colors** -- Use `NoirColors.of(context)` for ALL dynamic colors:
- Accent/Crimson: `NoirColors.crimson` (#B5363B) -- warm garnet, NOT cold blue-red
- Dark bg: #080808 / Light bg: #FCFBF9
- NO shadows anywhere -- use 6% opacity borders (`c.border`, `c.borderMedium`)
- Film grain overlay on both modes

**Typography** -- Three font families:
- `NoirTypography.sectionHeader` / `.displayLarge` / `.greetingName` -- Bebas Neue (uppercase, letterspaced)
- `NoirTypography.bodyLarge` / `.bodyMedium` / `.labelLarge` -- DM Sans (body, labels)
- `NoirTypography.numberLarge` / `.numberMedium` / `.numberSmall` -- Cormorant (metric numbers)

**Key files:**
- `lib/theme/colors.dart` -- `NoirColors` class with `.dark` and `.light` palettes
- `lib/theme/typography.dart` -- `NoirTypography` class
- `lib/theme/theme.dart` -- ThemeData configuration
- `lib/theme/theme_notifier.dart` -- Auto time-based light/dark switching
- `lib/utils/theme.dart` -- `AppTheme` bridge for unrewritten screens (maps old colors to Noir)

**Design Rules (from `.claude/rules/design-rules.md`):**
1. Backend integration is MANDATORY -- every screen uses real providers/API
2. Functionality MUST match original app -- every tap, navigation, interaction
3. ZERO mock data -- `MockData` class is FORBIDDEN
4. "VETTRA" branding -- never "STRAVA" or third-party names
5. Nav order: [Home] [Activities] RUN [Plans] [AI Coach]

### What "Redesigned" Means

A screen is REDESIGNED when:
1. It uses `NoirColors.of(context)` for ALL colors (background, text, borders, cards)
2. It uses `NoirTypography.*` for ALL text styles (headers = Bebas, body = DM Sans, numbers = Cormorant)
3. It has NO shadows -- only 6% opacity borders
4. It has NO hardcoded colors -- everything via `NoirColors`
5. It retains 100% of the functionality from the old app (same taps, same navigation, same data)
6. It uses the same Riverpod providers, ApiClient, and Firebase Auth as the old app
7. All interactive elements work identically to the old app
8. It looks premium, editorial, and intentional -- invoke the frontend-design skill for aesthetic decisions
9. It imports `'../theme/colors.dart'` and `'../theme/typography.dart'` -- NOT `'../utils/theme.dart'` (AppTheme)
10. It has NO emojis anywhere in the UI -- the user explicitly hates emojis in the interface

### Screens That Don't Exist Yet in the New App

If a screen exists in the old app but NOT in the new app:
1. COPY the old screen file to the new app's `lib/screens/` directory
2. Then convert it to Noir (same as any other unconverted screen)
3. Make sure its imports point to the new app's providers/models/services (these are already copied)

### Import Migration Checklist

When converting a screen, the sub-agent MUST:
- REMOVE: `import '../utils/theme.dart';` (old AppTheme bridge)
- ADD: `import '../theme/colors.dart';` (NoirColors)
- ADD: `import '../theme/typography.dart';` (NoirTypography)
- REPLACE: All `AppTheme.primaryAccent` -> `NoirColors.crimson`
- REPLACE: All `AppTheme.backgroundColor` -> `c.background` (where `c = NoirColors.of(context)`)
- REPLACE: All `AppTheme.textColor` -> `c.text`
- REPLACE: All `AppTheme.subtitleColor` -> `c.textMuted`
- REPLACE: All `AppTheme.cardColor` -> `c.cardBg`
- REPLACE: All `AppTheme.borderColor` -> `c.border`
- REPLACE: All `BoxShadow(...)` -> `Border.all(color: c.border)`

---

## FIRST: Check for Existing Session

Before doing anything else, check if `REDESIGN_TODO.md` exists in the new app root:

```bash
ls "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/REDESIGN_TODO.md" 2>/dev/null && echo "RESUME" || echo "NEW SESSION"
```

- **If it exists -> RESUME MODE**: Read `REDESIGN_TODO.md`, find the first unchecked screen, and continue from there. Do NOT restart from scratch.
- **If it doesn't exist -> NEW SESSION**: Proceed from Phase 0.

---

## Context Budget -- Token Discipline

You are designed to run for hours across multiple sessions. Context exhaustion is expected.

### Rules
- **Never read a full screen file to check if it's converted.** Use `Grep` for `NoirColors.of` or `AppTheme.` to determine status.
- **Never read files larger than 300 lines in full.** Use offset/limit.
- **Delegate all fixes to sub-agents** -- you are a coordinator.
- **Write findings to REDESIGN_TODO.md** -- don't carry them in context.
- After each screen conversion+verification, update REDESIGN_TODO.md and commit.
- If context is getting long: finish current screen, checkpoint, stop. Next session resumes.

---

## REDESIGN_TODO.md -- Persistent Session State

This file is your external memory. Update it constantly.

### Format

```markdown
# Vettra Redesign Marathon -- Session State

**Session started**: YYYY-MM-DD HH:MM
**Last updated**: YYYY-MM-DD HH:MM

## NEXT ACTION (resume here)
> [Exact next action -- e.g., "Convert activities_screen.dart to Noir aesthetic"]

## Screen Conversion Status
<!-- C = Converted, V = Verified with Playwright, F = Functional parity confirmed -->
| # | Screen File | C | V | F | Notes |
|---|------------|---|---|---|-------|
| 1 | home_feed.dart | x | x | x | Done in prior session |
| 2 | activities_screen.dart | x | x | x | Done in prior session |
| 3 | activity_detail_screen.dart | x | | | Converted, needs Playwright test |
| 4 | ai_trainer_screen.dart | | | | Not started |
...

## Sub-Agents Spawned
| Screen | Task ID | Status | Notes |
|--------|---------|--------|-------|
| activity_detail_screen.dart | [id] | IN_PROGRESS | Converting to Noir |

## Issues Found
| Screen | Issue | Status | Fix Agent |
|--------|-------|--------|-----------|
| home_feed.dart | Recovery ring not showing | FIXED | sub-001 |
```

---

## Phase 0 -- Project Intelligence & Inventory

### 0.1 Read Design System
1. Read `lib/theme/colors.dart` -- understand all NoirColors tokens
2. Read `lib/theme/typography.dart` -- understand all NoirTypography styles
3. Read `.claude/rules/design-rules.md` -- absolute design rules
4. Read any CLAUDE.md files

### 0.2 Build Screen Inventory

Compare old app screens vs new app screens to determine conversion status:

```bash
# Old app screens
ls "/Users/jakobwredstrom/Desktop/Vetra App/vettra-app/lib/screens/"

# New app screens
ls "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/lib/screens/"
```

### 0.3 Determine Conversion Status

For each screen in the new app, check if it's already converted to Noir:

```bash
# If it uses NoirColors.of(context), it's converted
grep -l "NoirColors.of" "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/lib/screens/"*.dart

# If it uses AppTheme. or hardcoded colors, it's NOT converted
grep -l "AppTheme\." "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/lib/screens/"*.dart
```

### 0.4 Priority Order

Convert screens in this order (most visible first):
1. **Home/Feed** -- `home_feed.dart`, `feed_screen.dart`
2. **Activities** -- `activities_screen.dart`, `activity_detail_screen.dart`
3. **Plans** -- `training_plans_screen.dart`, `plans_screen.dart`
4. **AI Coach** -- `ai_trainer_screen.dart`, `ai_coach_screen.dart`
5. **Profile & Settings** -- `profile_screen.dart`, `settings_screen.dart`, `personal_info_screen.dart`
6. **Health** -- `health_screen.dart`, `health_insights_screen.dart`, `fitness_dashboard_screen.dart`
7. **Knowledge** -- `knowledge_hub_screen.dart`, `knowledge_article_screen.dart`
8. **Races** -- `races_screen.dart`, `race_detail_screen.dart`, `race_edit_screen.dart`
9. **Workouts** -- `workout_builder_screen.dart`, `workout_execution_screen.dart`, `record_workout_screen.dart`
10. **Auth/Onboarding** -- `auth_screen.dart`, `onboarding_screen.dart`, `onboarding_splash_screen.dart`
11. **Everything else** -- remaining screens

### 0.5 Write REDESIGN_TODO.md

Write the full inventory to `REDESIGN_TODO.md` with status for each screen. Commit immediately.

---

## Phase 1 -- Environment Setup

### 1.1 Start Both Apps

Start BOTH the old and new apps simultaneously:

```bash
# Start OLD app on port 3006 (reference)
cd "/Users/jakobwredstrom/Desktop/Vetra App/vettra-app" && flutter run -d chrome --web-port=3006 --dart-define=USE_PRODUCTION_URL=true --dart-define=DEV_EMAIL=devqa@segments.app "--dart-define=DEV_PASSWORD=SegmentsDevQA2024!" &

# Start NEW app on port 3005 (target)
cd "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend" && flutter run -d chrome --web-port=3005 --dart-define=USE_PRODUCTION_URL=true --dart-define=DEV_EMAIL=devqa@segments.app "--dart-define=DEV_PASSWORD=SegmentsDevQA2024!" &
```

Use Desktop Commander (`mcp__Desktop_Commander__start_process`) for these -- it allows you to interact with the process later for hot reload.

### 1.2 Verify Both Apps Load

Use Playwright to navigate to both ports and verify they load:
- `http://localhost:3005` -- new app
- `http://localhost:3006` -- old app

If either shows the onboarding/login screen, click through it using the dev credentials.

### 1.3 Flutter Hot Reload Strategy

After editing files in the new app, trigger hot reload:
1. Use `mcp__Desktop_Commander__interact_with_process` to send `r` to the Flutter process
2. OR use `mcp__Control_your_Mac__osascript` to send keystrokes to the terminal
3. If neither works, restart the Flutter process

IMPORTANT: Flutter web in debug mode uses CanvasKit. Screenshots work but the accessibility tree (Playwright snapshot) may be empty. Use coordinate-based clicks for Flutter web.

---

## Phase 2 -- Screen-by-Screen Conversion (THE MAIN LOOP)

For EACH unconverted screen, follow this exact procedure:

### Step 2.0 -- Pre-Check: Does the Screen Exist? Are Its Widgets Converted?

**A. Check if screen exists in new app:**
```bash
ls "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/lib/screens/[screen_name].dart" 2>/dev/null
```
If it does NOT exist, copy it from the old app first:
```bash
cp "/Users/jakobwredstrom/Desktop/Vetra App/vettra-app/lib/screens/[screen_name].dart" "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/lib/screens/"
```

**B. Check which widgets this screen depends on:**
```bash
grep "import.*widgets/" "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/lib/screens/[screen_name].dart"
```
For each imported widget, check if it's already converted:
```bash
grep -c "NoirColors.of" "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/lib/widgets/[widget_name].dart"
```
If a widget is NOT converted (returns 0), spawn a widget conversion sub-agent FIRST and wait for it before converting the screen.

### Step 2.1 -- Read the Old Screen (Reference)

```
Read the old app's version of the screen:
"/Users/jakobwredstrom/Desktop/Vetra App/vettra-app/lib/screens/[screen_name].dart"
```

Note:
- Every widget, every tap handler, every navigation target
- Every provider it uses (`ref.watch`, `ref.read`)
- Every API call and data model
- Every interactive element (buttons, cards, swipes, gestures)
- The overall layout structure

### Step 2.2 -- Read the New Screen (Current State)

```
Read the new app's version:
"/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/lib/screens/[screen_name].dart"
```

Determine:
- Is it using `NoirColors.of(context)` or hardcoded colors?
- Is it using `NoirTypography.*` or inline TextStyles?
- Does it have all the functionality of the old screen?
- Are there any `MockData` references? (FORBIDDEN)
- Are shadows present? (MUST be removed)

### Step 2.3 -- Invoke Frontend-Design Skill for Aesthetic Decisions

**CRITICAL: Always invoke the `frontend-design` skill before making aesthetic decisions for a screen that hasn't been designed yet.**

Use the Skill tool:
```
Skill: frontend-design
```

The frontend-design skill provides:
- Typography choices (which NoirTypography style for what)
- Color usage (when to use crimson accent, when muted, when surface colors)
- Spatial composition (padding, spacing, card layout)
- Motion/transitions (if applicable)
- How to make the screen feel premium and intentional

You already have the Noir design system. The frontend-design skill helps you apply it with taste and intention -- not mechanically.

### Step 2.4 -- Spawn Conversion Sub-Agent

**ALWAYS delegate the actual code changes to a sub-agent.** This preserves your context window.

```
Agent tool:
  subagent_type: general-purpose
  run_in_background: true
  isolation: worktree
  prompt: |
    You are a Vettra Redesign Sub-Agent. Convert one screen to the Noir aesthetic.

    PROJECT ROOT: /Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend
    SCREEN TO CONVERT: lib/screens/[screen_name].dart

    REFERENCE (old screen -- copy ALL functionality from this):
    [paste key excerpts or full content of old screen showing providers, navigation, taps]

    DESIGN SYSTEM:
    - Colors: Use `NoirColors.of(context)` for ALL colors. Import from '../theme/colors.dart'
    - Typography: Use `NoirTypography.*` for ALL text. Import from '../theme/typography.dart'
      - Section headers: NoirTypography.sectionHeader (Bebas Neue, uppercase, letterspaced)
      - Body text: NoirTypography.bodyMedium / .bodyLarge (DM Sans)
      - Numbers/metrics: NoirTypography.numberLarge / .numberMedium (Cormorant serif)
      - Labels: NoirTypography.labelLarge / .labelMedium (DM Sans, uppercase)
      - Meta/detail: NoirTypography.meta (DM Sans italic)
    - NO shadows -- use borders: Border.all(color: c.border) or c.borderMedium
    - Card backgrounds: c.cardBg or c.surface2
    - Page background: c.background
    - Accent color: NoirColors.crimson (for CTAs, highlights, active states)
    - Text colors: c.text (primary), c.textMuted (secondary), c.textFaint (tertiary)

    ABSOLUTE RULES:
    1. EVERY tap, button, navigation from the old screen MUST work identically
    2. Use the SAME Riverpod providers as the old screen
    3. NO MockData -- use real providers with AsyncValue.when(loading:, error:, data:)
    4. NO shadows -- 6% opacity borders only
    5. NO hardcoded colors -- everything via NoirColors.of(context)
    6. Replace any "STRAVA" text with "VETTRA"
    7. Maintain ConsumerStatefulWidget / ConsumerWidget pattern
    8. NO EMOJIS in the UI -- the user explicitly hates emojis. Remove any emoji from display text.
    9. REMOVE `import '../utils/theme.dart'` (old AppTheme) -- replace with `import '../theme/colors.dart'` and `import '../theme/typography.dart'`
    10. All `AppTheme.xxx` references must be replaced with `NoirColors` / `NoirTypography` equivalents

    AESTHETIC GUIDANCE:
    [Insert specific aesthetic notes from frontend-design skill invocation]

    STEPS:
    1. Read the current new screen file
    2. Read the old screen file for functional reference
    3. Rewrite the screen with Noir aesthetic while preserving ALL functionality
    4. Run: flutter analyze lib/screens/[screen_name].dart
    5. If 0 errors: git add lib/screens/[screen_name].dart && git commit -m "design: convert [screen_name] to Noir aesthetic"
    6. Report: "CONVERTED: [screen_name].dart" or "FAILED: [reason]"
```

### Step 2.5 -- Verify with Playwright (After Sub-Agent Completes)

After the conversion sub-agent reports back, verify the screen with Playwright.

**IMPORTANT: Playwright controls ONE browser page at a time.** To compare old and new apps:
- Navigate to port 3005 (new app), test, take screenshot
- Then navigate to port 3006 (old app), test, take screenshot
- Compare results mentally (you can see both screenshots)

**IMPORTANT: After code changes, restart the Flutter process.** Flutter web hot reload is unreliable. Kill the old process and start a new one:
```bash
# Kill old process, start new
kill [PID] && cd "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend" && flutter run -d chrome --web-port=3005 ...
```
Wait 6-8 seconds after navigation for Flutter to fully render before taking screenshots.

#### A. Visual Check -- USE FRONTEND-DESIGN SKILL TO EVALUATE

1. Navigate Playwright to `http://localhost:3005`
2. Navigate to the converted screen (click through nav)
3. Take a screenshot with `browser_take_screenshot`
4. **Invoke the `frontend-design` skill to evaluate the screenshot:**
   - Does it look premium and intentional?
   - Are the colors correct (dark bg, cream text, crimson accents)?
   - Is the typography hierarchy right (Bebas headers, DM Sans body, Cormorant numbers)?
   - Are there any jarring elements (white cards on dark bg, bright colors that don't belong)?
   - Does the spatial composition feel balanced and generous?
   - Are there any leftover old-design elements (shadows, wrong fonts, hardcoded colors)?
5. If the frontend-design evaluation finds issues -> SPAWN FIX SUB-AGENT with specific aesthetic fixes

#### B. Functional Parity Check (New App -- port 3005)
1. Navigate to the converted screen
2. Click every interactive element (cards, buttons, rows, tabs) using coordinate-based clicks
3. After each click, take a screenshot to verify the navigation target loaded correctly
4. Verify data is loading from API (text visible, not blank, not "Loading..." stuck)
5. Check: do tappable cards navigate to the right detail screens?
6. Check: do back buttons work?
7. Check: do tabs switch content correctly?

#### C. Compare with Old App (port 3006)
1. Navigate Playwright to `http://localhost:3006`
2. Navigate to the SAME screen in the old app
3. Take a screenshot for reference
4. Click the SAME interactive elements in the same order
5. Verify IDENTICAL behavior: same navigation targets, same data shown, same number of items
6. If behavior differs -> log the difference and SPAWN FIX SUB-AGENT

#### D. Check Console for Errors
Use `browser_console_messages` or check the Flutter process output for:
- Dart assertion errors (e.g., `Assertion failed`, `RangeError`)
- API errors (e.g., `HTTP 500`, `Failed to load resource`)
- Rendering errors (e.g., `EXCEPTION CAUGHT BY WIDGETS LIBRARY`)
- Widget overflow errors (e.g., `RenderFlex overflowed`)

Any error = SPAWN FIX SUB-AGENT immediately.

#### E. Run `flutter analyze` on the Whole Project (every 3-5 screens)

```bash
cd "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend" && flutter analyze 2>&1 | tail -20
```

If there are errors in ANY file (not just the screen you converted), spawn a fix sub-agent.

### Step 2.6 -- Update REDESIGN_TODO.md

Mark the screen as:
- `C` = Converted (code changed)
- `V` = Visually verified (Playwright screenshot looks correct)
- `F` = Functionally verified (all taps/navigation work, matches old app)

Commit the update.

### Step 2.7 -- Move to Next Screen

Repeat Steps 2.1-2.6 for the next unconverted screen. NEVER stop.

---

## Phase 3 -- Widget Conversion (Inline + Catch-Up)

Widgets are converted in TWO ways:

### 3A. Inline During Screen Conversion (preferred)
When converting a screen, check which widgets it imports. If any widget still uses `AppTheme`, convert it FIRST (spawn a sub-agent for the widget before the screen). The screen conversion sub-agent should reference the already-converted widget.

Before converting any screen, run:
```bash
# Find which widgets the screen imports
grep "import.*widgets/" "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/lib/screens/[screen_name].dart"
# Then check if those widgets are converted
grep -l "AppTheme\." [each widget file found above]
```

### 3B. Catch-Up Sweep (after all screens)
After all screens are done, sweep for any remaining unconverted widgets:

```bash
grep -rl "AppTheme\." "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/lib/widgets/"
```

For each unconverted widget, spawn a sub-agent (same pattern as screens).

Key widgets to prioritize if not yet converted:
- `bottom_nav.dart` / `bottom_nav_scaffold.dart` -- navigation (likely already done)
- `morning_checkin_card.dart` -- home screen
- `today_workout_card.dart` -- home screen
- `activity_charts.dart` -- activity detail (chart colors must use NoirColors)
- `training_plan_preview.dart` -- plans
- `chat_message_widget.dart` -- AI coach
- `navigation_drawer.dart` / `nav_drawer.dart` -- side menu

---

## Phase 4 -- Full App Walkthrough (Post-Conversion QA)

After ALL screens are converted, do a complete app walkthrough with Playwright:

### 4.1 Navigate Every Tab
1. Home -> verify content loads, cards are tappable
2. Activities -> verify list loads, items navigate to detail
3. Plans -> verify plan loads or empty state shows
4. AI Coach -> verify chat interface loads

### 4.2 Test Key User Flows
1. **View activity detail**: Home -> tap activity card -> verify detail screen loads with charts, stats, map
2. **Navigate to races**: Home -> tap race card -> verify races screen
3. **Morning check-in**: Home -> interact with check-in card -> verify it submits
4. **View profile**: Hamburger menu -> Profile -> verify settings load
5. **Knowledge hub**: Navigate to knowledge -> tap article -> verify article renders

### 4.3 Compare Side-by-Side with Old App
For each flow above, do the same in the old app (port 3006) and verify identical behavior.

---

## Phase 5 -- QA Marathon

When all screens are converted and the walkthrough passes, invoke the QA Marathon skill:

```
Skill: qa-marathon
```

This runs the full autonomous QA session across the entire new app, finding any remaining bugs, crashes, or issues.

---

## Phase 6 -- Final Full Click-Through

After QA Marathon completes, do one final comprehensive click-through:

### 6.1 Start Both Apps
Ensure both old (3006) and new (3005) are running.

### 6.2 Systematic Click-Through
Navigate through EVERY screen in the new app:
1. Take a screenshot of each screen
2. Click every button, card, tab, and interactive element
3. Verify navigation works
4. Compare behavior with old app on port 3006
5. Log any discrepancies to REDESIGN_TODO.md

### 6.3 Final Report
Write a `REDESIGN_COMPLETE.md` in the project root:
```markdown
# Vettra Redesign -- Completion Report

**Completed**: YYYY-MM-DD HH:MM
**Screens converted**: N/N
**Screens verified**: N/N
**Issues found and fixed**: N
**Remaining issues**: N (list)

## Screen-by-Screen Status
[full table from REDESIGN_TODO.md]

## Known Limitations
[any screens that couldn't be fully converted and why]
```

---

## Multi-Agent Operating Model

### Conversion Sub-Agent (one per screen)
- Receives: screen file path, old screen reference, design system rules
- Makes the conversion (reads old, rewrites new with Noir)
- Runs `flutter analyze`
- Commits the change
- Reports: CONVERTED or FAILED

### Fix Sub-Agent (one per bug)
- Receives: specific bug location and description
- Makes minimal fix
- Runs `flutter analyze`
- Commits and reports

### Quick-Spawn Template for Fix Sub-Agent

```
Agent tool:
  subagent_type: general-purpose
  run_in_background: true
  isolation: worktree
  prompt: |
    You are a Fix Sub-Agent for the Vettra redesign.
    PROJECT: /Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend

    BUG:
    - File: [path]
    - Description: [what's wrong]
    - Expected: [what should happen]
    - Fix: [suggested change]

    STEPS:
    1. Read the file
    2. Make the minimal fix
    3. Run: flutter analyze [file]
    4. git add [file] && git commit -m "fix: [description]"
    5. Report: FIXED or FAILED

    RULES: Never ask questions. Minimal changes only.
```

---

## Flutter Web + Playwright Testing Notes

Flutter web renders to a canvas (CanvasKit mode). This means:

1. **Playwright's accessibility snapshot may be empty** -- Flutter has its own semantics layer
2. **Use coordinate-based clicks** (`page.mouse.click(x, y)`) for interacting with Flutter web
3. **Screenshots DO work** -- use `browser_take_screenshot` to capture the canvas
4. **Wait for Flutter to load** -- Flutter web takes 3-8 seconds to initialize. After navigation, wait at least 5 seconds before screenshotting.
5. **Enable semantics** -- click the `flt-semantics-placeholder` button if visible (or dismiss it)
6. **Hot reload** -- after editing files, either:
   - Send `r` to the Flutter process stdin via Desktop Commander
   - Reload the browser page (may need full page reload for web)
   - Restart the Flutter process if hot reload doesn't pick up changes

### Playwright Testing Pattern for Flutter Web

```javascript
// Navigate
await page.goto('http://localhost:3005');
await page.waitForTimeout(6000); // Wait for Flutter

// Screenshot
await page.screenshot({ path: 'screen.png', type: 'png' });

// Click by coordinates (find from screenshot)
await page.mouse.click(250, 400);
await page.waitForTimeout(1000);

// Scroll
await page.mouse.wheel(0, 300);

// Type (after clicking a text field)
await page.keyboard.type('some text');
```

### Login Flow for Playwright

If the app shows the onboarding/login screen:
1. Click through splash pages ("Get started")
2. Click "Dev login" at the bottom of the login screen
3. Enter password in the modal: `SegmentsDevQA2024!`
4. Click "Sign in"
5. If onboarding appears, click through all steps (select defaults, click NEXT)

---

## Screen Inventory (Complete List)

### Old App Screens (51 files in vettra-app/lib/screens/)
```
active_workout_screen.dart
activities_screen.dart
activity_detail_screen.dart
ai_trainer_screen.dart
audio_settings_screen.dart
auth_screen.dart
coach_evaluation_wizard_screen.dart
connect_device_screen.dart
connections_screen.dart
data_processing_screen.dart
deals_screen.dart
developer_connections_screen.dart
feed_screen.dart
fitness_dashboard_screen.dart
gear_deals_screen.dart
health_insights_screen.dart
health_screen.dart
injury_log_screen.dart
intelligence_reveal_screen.dart
knowledge_article_screen.dart
knowledge_hub_screen.dart
legal_screen.dart
morning_checkin_screen.dart
notification_settings_screen.dart
onboarding_complete_screen.dart
onboarding_screen.dart
onboarding_splash_screen.dart
paywall_screen.dart
performance_screen.dart
personal_info_screen.dart
personal_records_screen.dart
plan_generation_screen.dart
plan_reveal_screen.dart
plan_wizard_screen.dart
profile_screen.dart
race_detail_screen.dart
race_edit_screen.dart
races_screen.dart
record_workout_screen.dart
running_profile_screen.dart
settings_screen.dart
shoe_deals_screen.dart
squad_detail_screen.dart
squad_invite_screen.dart
squad_screen.dart
strength_glossary_screen.dart
training_plans_screen.dart
whoop_screen.dart
workout_builder_screen.dart
workout_execution_screen.dart
workout_library_screen.dart
workout_summary_screen.dart
```

### New App Additional Screens (in vettra-newfrontend but not in old)
```
home_feed.dart (replaces feed_screen.dart as the Noir home page)
app_shell.dart (main app shell with bottom nav)
ai_coach_screen.dart (Noir version of AI trainer)
plans_screen.dart (Noir version of training plans)
```

### Shared Widgets to Convert
```
bottom_nav.dart / bottom_nav_scaffold.dart
morning_checkin_card.dart
today_workout_card.dart
activity_charts.dart
activity_laps_table.dart
activity_splits_table.dart
activity_map.dart
activity_analysis.dart
best_efforts_list.dart
segment_efforts_list.dart
training_plan_preview.dart
chat_message_widget.dart
phase_timeline.dart
race_countdown_widget.dart
week_card.dart
week_edit_sheet.dart
weekly_progress_bar.dart
knowledge_article_card.dart
knowledge_category_card.dart
knowledge_chat_sheet.dart
knowledge_daily_insight_card.dart
health_status_card.dart
readiness_score_badge.dart
shimmer_loading.dart
user_avatar.dart
navigation_drawer.dart / nav_drawer.dart
difficulty_badge.dart
film_grain.dart (already Noir-specific)
```

---

## Operating Principles

**Never stop early.** If there are 50 screens, convert all 50. If Playwright finds a bug, spawn a fix agent and keep going.

**Never ask questions.** Read the old app's code to understand intent. Read the design system to understand aesthetics. If truly ambiguous, use your best judgment.

**You are a coordinator, not a lone worker.** Spawn sub-agents for all code changes. Your job is to read, plan, verify, and coordinate.

**Spawn sub-agents immediately.** Don't accumulate screens to convert later. Convert each screen as you reach it.

**Always verify with Playwright.** A screen is not done until you've taken a screenshot AND clicked through it.

**Always compare with the old app.** The old app on port 3006 is the functional truth. The new app must behave identically.

**Invoke frontend-design skill for aesthetic decisions.** You have the design system (colors, typography). The frontend-design skill tells you HOW to apply them with taste.

**Preserve all functionality.** You are changing DESIGN ONLY -- colors, fonts, layout, spacing, visual styling. Every provider, every API call, every navigation target stays the same.

**Context checkpoint frequently.** After every 2-3 screens, update REDESIGN_TODO.md and commit. If context is getting long, stop gracefully -- the next session will resume.

---

## When to Stop

You stop when ALL of these are true:
1. Every screen in the inventory is marked C+V+F (Converted, Visually verified, Functionally verified)
2. Every shared widget is converted to Noir
3. QA Marathon has been run (Phase 5)
4. Final click-through completed (Phase 6)
5. `REDESIGN_COMPLETE.md` written and committed
6. No outstanding crashes or assertion errors in the app

**Context window exhaustion is expected.** When context is getting long:
1. Finish the current screen verification
2. Update REDESIGN_TODO.md with exact next action
3. Commit
4. Stop -- the next `/vettra-redesign-marathon` invocation resumes automatically
