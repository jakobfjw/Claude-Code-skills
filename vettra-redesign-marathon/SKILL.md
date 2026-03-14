---
name: vettra-redesign-marathon
description: |
  Fully autonomous, never-stopping redesign marathon for the Vettra running app. Redesigns every screen from the old app (vettra-app) into the new Noir aesthetic (vettra-newfrontend), screen by screen, with Playwright-based visual and functional testing after each screen. Use this skill when the user says /vettra-redesign-marathon, "redesign everything", "convert all screens", "run the redesign", "redesign marathon", or any variation suggesting they want the full autonomous redesign pipeline. Delegates ALL work to sub-agents to preserve coordinator context. Runs QA marathon when all screens are done. NEVER stops. NEVER asks questions. Makes ALL decisions autonomously. Fully contextual for fresh agent sessions.
---

# Vettra Redesign Marathon -- Autonomous Screen-by-Screen Conversion

You are the **Redesign Coordinator**. Your ONLY job: **dispatch sub-agents, track results, update state.** You are a dispatcher -- not a reader, not a coder, not a designer. You never read screen files. You never take screenshots. You never write Dart code. You delegate everything.

---

## Architecture -- Three-Tier Agent Model

```
TIER 1: COORDINATOR (you)
  - Reads ONLY: REDESIGN_TODO.md, grep output, agent results
  - NEVER reads: screen files, widget files, design system files
  - NEVER takes: Playwright screenshots
  - NEVER writes: Dart code
  - NEVER invokes: frontend-design skill (sub-agents do this)
  - Budget: ~200 tokens per screen cycle
  - Job: dispatch -> wait -> record result -> next

TIER 2: CONVERSION AGENTS (one per screen or widget)
  - Self-contained: gets FULL design system + old screen content in prompt
  - Reads old screen, reads new screen, converts, runs flutter analyze, commits
  - Invokes frontend-design skill for aesthetic decisions
  - Reports: CONVERTED or FAILED + reason

TIER 3: VERIFICATION AGENTS (one per screen, after conversion)
  - Uses Playwright MCP to test the converted screen
  - Takes screenshots, clicks through elements, checks for crashes
  - Compares with old app on port 3006
  - Reports: VERIFIED or ISSUES_FOUND + list

TIER 4: FIX AGENTS (one per bug)
  - Minimal targeted fix, flutter analyze, commit
  - Reports: FIXED or FAILED
```

**Why this works:** Each sub-agent gets a FRESH context window. The coordinator's context stays tiny because it never reads files or processes screenshots. A 50-screen conversion costs the coordinator ~10K tokens total, leaving massive headroom for tracking and orchestration.

---

## CRITICAL CONTEXT -- Embedded in Every Agent Prompt

### Project Paths

- **OLD APP**: `/Users/jakobwredstrom/Desktop/Vetra App/vettra-app/`
- **NEW APP**: `/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/`
- **Package name**: `concept_e_noir`
- **Brand guide**: `/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/BRAND.md` -- authoritative design reference
- **CLAUDE.md**: `/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/CLAUDE.md` -- project architecture + rules

### Run Commands

```bash
# OLD app on port 3006 (reference)
cd "/Users/jakobwredstrom/Desktop/Vetra App/vettra-app" && flutter run -d chrome --web-port=3006 --dart-define=USE_PRODUCTION_URL=true --dart-define=DEV_EMAIL=devqa@segments.app "--dart-define=DEV_PASSWORD=SegmentsDevQA2024!"

# NEW app on port 3005 (target)
cd "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend" && flutter run -d chrome --web-port=3005 --dart-define=USE_PRODUCTION_URL=true --dart-define=DEV_EMAIL=devqa@segments.app "--dart-define=DEV_PASSWORD=SegmentsDevQA2024!"
```

### Dev Credentials
- Email: `devqa@segments.app`
- Password: `SegmentsDevQA2024!`
- Production API: `https://segmentsapp-102258610874.europe-west1.run.app/api`
- ALWAYS use `--dart-define=USE_PRODUCTION_URL=true`

---

## DESIGN SYSTEM REFERENCE (embed in every conversion agent prompt)

Paste this ENTIRE block into every conversion agent prompt. Do NOT tell agents to "read colors.dart" -- that wastes their tokens on file I/O.

```
=== NOIR DESIGN SYSTEM -- COMPACT REFERENCE ===

COLORS -- Use `final c = NoirColors.of(context);` then access via `c.xxx`
Import: `import '../theme/colors.dart';`

Dynamic tokens (adapt to light/dark mode):
  c.background    -- Dark: #080808 / Light: #FCFBF9
  c.surface       -- Dark: #0F0F0F / Light: #F7F5F2
  c.surface2      -- Dark: #171717 / Light: #FFFFFF
  c.surface3      -- Dark: #1F1F1F / Light: #F2F0ED
  c.cardBg        -- Dark: #171717 / Light: #FFFFFF
  c.text          -- Dark: #F2EDE4 / Light: #0A0A0A
  c.textMuted     -- Dark: #666666 / Light: #6B6B6B
  c.textFaint     -- Dark: #2E2E2E / Light: #BBBBBB
  c.subtitle      -- Dark: #666666 / Light: #6B6B6B
  c.accent        -- Dark: #F2EDE4 / Light: #0A0A0A
  c.secondary     -- Dark: #2E2E2E / Light: #E8E6E3
  c.border        -- Dark: rgba(255,255,255,0.05) / Light: rgba(0,0,0,0.06)
  c.borderMedium  -- Dark: rgba(255,255,255,0.06) / Light: rgba(0,0,0,0.06)
  c.borderStrong  -- Dark: rgba(255,255,255,0.10) / Light: rgba(0,0,0,0.10)
  c.navBg         -- Dark: #080808 / Light: #FCFBF9

Static tokens (same in both modes):
  NoirColors.crimson      -- #B5363B (warm garnet -- primary accent, CTAs)
  NoirColors.crimsonDark  -- #9A2F34
  NoirColors.crimsonDim   -- #B5363B at 15% opacity
  NoirColors.crimsonGlow  -- #B5363B at 12% opacity
  NoirColors.hrvColor     -- #4A8C5C (green for HRV)
  NoirColors.hrColor      -- #B5363B (crimson for HR)
  NoirColors.sleepColor   -- #666666

TYPOGRAPHY -- Use `NoirTypography.xxx`
Import: `import '../theme/typography.dart';`

  Headers (Bebas Neue -- uppercase, letterspaced):
    NoirTypography.displayLarge   -- 56px, w400
    NoirTypography.displayMedium  -- 42px, w400
    NoirTypography.greetingName   -- 48px, w400
    NoirTypography.sectionHeader  -- 16px, w400

  Body (DM Sans):
    NoirTypography.headlineLarge  -- 20px, w500
    NoirTypography.headlineMedium -- 18px, w500
    NoirTypography.headlineSmall  -- 16px, w500
    NoirTypography.displaySmall   -- 24px, w600
    NoirTypography.subheadline    -- 17px, w400
    NoirTypography.bodyLarge      -- 16px, w300
    NoirTypography.bodyMedium     -- 14px, w400
    NoirTypography.bodySmall      -- 12px, w400
    NoirTypography.meta           -- 13px, w400, italic
    NoirTypography.eyebrow        -- 13px, w600
    NoirTypography.greeting       -- 13px, w300

  Labels (DM Sans -- uppercase, letterspaced):
    NoirTypography.labelLarge     -- 14px, w600
    NoirTypography.labelMedium    -- 12px, w600
    NoirTypography.labelSmall     -- 11px, w500

  Numbers (Cormorant -- serif):
    NoirTypography.numberLarge    -- 40px, w700
    NoirTypography.numberMedium   -- 28px, w600
    NoirTypography.numberSmall    -- 20px, w600
    NoirTypography.numberXSmall   -- 15px, w500

ABSOLUTE RULES:
  1. NO shadows anywhere -- use Border.all(color: c.border) or c.borderMedium
  2. NO hardcoded colors -- everything via NoirColors.of(context) or NoirColors.xxx
  3. NO AppTheme references -- remove import '../utils/theme.dart' entirely
  4. NO emojis in the UI -- user explicitly hates emojis
  5. NO MockData -- use real Riverpod providers with AsyncValue.when()
  6. Functionality MUST match the old screen exactly (same taps, navigation, data)
  7. Use same Riverpod providers, ApiClient, Firebase Auth as old app
  8. "VETTRA" branding only -- never "STRAVA" or third-party names
  9. Nav order: [Home] [Activities] RUN [Plans] [AI Coach]
  10. Card style: c.cardBg background, Border.all(color: c.border), BorderRadius.circular(16)

IMPORT MIGRATION:
  REMOVE: import '../utils/theme.dart';
  ADD:    import '../theme/colors.dart';
  ADD:    import '../theme/typography.dart';
  REPLACE: AppTheme.primaryAccent -> NoirColors.crimson
  REPLACE: AppTheme.backgroundColor -> c.background
  REPLACE: AppTheme.textColor -> c.text
  REPLACE: AppTheme.subtitleColor -> c.textMuted
  REPLACE: AppTheme.cardColor -> c.cardBg
  REPLACE: AppTheme.borderColor -> c.border
  REPLACE: BoxShadow(...) -> Border.all(color: c.border)
  REPLACE: Colors.xxx (any Material color) -> appropriate c.xxx token
=== END DESIGN SYSTEM ===
```

---

## FIRST: Check for Existing Session

```bash
cat "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/REDESIGN_TODO.md" 2>/dev/null && echo "RESUME" || echo "NEW SESSION"
```

- **RESUME**: Read REDESIGN_TODO.md, find `## NEXT ACTION`, execute it. Do NOT restart.
- **NEW SESSION**: Proceed from Phase 0.

---

## Token Discipline -- The Coordinator's Survival Rules

You are designed to run for hours across multiple sessions. Context exhaustion is expected. These rules keep you alive:

### NEVER do these (they kill your context):
- NEVER read a screen file (old or new) -- sub-agents read them
- NEVER read colors.dart, typography.dart, or design-rules.md -- they're embedded above
- NEVER take Playwright screenshots -- verification agents do that
- NEVER invoke the frontend-design skill -- conversion agents do that
- NEVER paste file contents into your own context -- delegate to agents
- NEVER write Dart code -- sub-agents write all code

### ALWAYS do these (they keep you lean):
- Use `grep -c "NoirColors.of" [file]` to check conversion status (returns count, not content)
- Use `grep -c "AppTheme\." [file]` to check if unconverted (returns count, not content)
- Read ONLY REDESIGN_TODO.md for state (never more than ~100 lines)
- Write agent results directly to REDESIGN_TODO.md (don't carry them in memory)
- Checkpoint (update TODO + commit) after every 3 screens
- After processing 8-10 screens: STOP, checkpoint, let next session continue

### Context exhaustion protocol:
1. Finish current screen's agent result processing
2. Update REDESIGN_TODO.md with exact next action
3. Write current timestamp to "Last updated"
4. Commit: `git add REDESIGN_TODO.md && git commit -m "chore: redesign checkpoint"`
5. STOP. The next `/vettra-redesign-marathon` session resumes automatically.

---

## REDESIGN_TODO.md -- Persistent State File

This file is your external brain. A new session reads this to resume instantly.

### Format

```markdown
# Vettra Redesign Marathon -- Session State

**Session started**: YYYY-MM-DD HH:MM
**Last updated**: YYYY-MM-DD HH:MM

## NEXT ACTION (resume here)
> [Exact imperative action -- e.g., "Spawn conversion agent for feed_screen.dart"]

## Phase Checklist
- [ ] Phase 0 -- Inventory + Environment
- [ ] Phase 1 -- Widget Conversion (batch)
- [ ] Phase 2 -- Screen Conversion (sequential)
- [ ] Phase 3 -- Playwright Deep Verification
- [ ] Phase 4 -- QA Marathon
- [ ] Phase 5 -- Final Click-Through + Report

## Widget Conversion Status
| # | Widget File | Status | Notes |
|---|------------|--------|-------|
| 1 | accent_headline.dart | CONVERTED | |
| 2 | activity_analysis.dart | PENDING | |

## Screen Conversion Status
<!-- S=Skipped(already done), C=Converted, V=Verified, F=Failed -->
| # | Screen File | Status | Notes |
|---|------------|--------|-------|
| 1 | home_feed.dart | S | Already Noir |
| 2 | feed_screen.dart | PENDING | |

## Issues Log
| Screen/Widget | Issue | Status | Fix Agent |
|---------------|-------|--------|-----------|
| feed_screen.dart | Missing provider import | FIXED | agent-001 |

## Progress Log (human-readable)
[YYYY-MM-DD HH:MM] Session started -- X widgets, Y screens to convert
[YYYY-MM-DD HH:MM] Widget batch complete: N/N converted
[YYYY-MM-DD HH:MM] Screen 1: feed_screen.dart -- CONVERTED + VERIFIED
```

---

## Phase 0 -- Inventory + Environment Setup

### 0.1 Quick Status Scan

Run these grep commands to determine what's already converted vs unconverted. Do NOT read any files.

```bash
# Converted screens (have NoirColors.of)
grep -rl "NoirColors.of" "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/lib/screens/" | xargs -I{} basename {} | sort

# Unconverted screens (have AppTheme)
grep -rl "AppTheme\." "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/lib/screens/" | xargs -I{} basename {} | sort

# Screens with hardcoded Material colors (Colors.xxx)
grep -rl "Colors\." "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/lib/screens/" | xargs -I{} basename {} | sort

# Same for widgets
grep -rl "AppTheme\.\|Colors\." "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/lib/widgets/" | xargs -I{} basename {} | sort
```

### 0.2 Write REDESIGN_TODO.md

Create the full inventory with every screen and widget listed. Mark already-converted items as `S` (skip). Commit immediately.

### 0.3 Start Both Apps

Use Desktop Commander (`start_process`) to launch both apps:

```bash
# Start OLD app on port 3006
cd "/Users/jakobwredstrom/Desktop/Vetra App/vettra-app" && flutter run -d chrome --web-port=3006 --dart-define=USE_PRODUCTION_URL=true --dart-define=DEV_EMAIL=devqa@segments.app "--dart-define=DEV_PASSWORD=SegmentsDevQA2024!"

# Start NEW app on port 3005
cd "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend" && flutter run -d chrome --web-port=3005 --dart-define=USE_PRODUCTION_URL=true --dart-define=DEV_EMAIL=devqa@segments.app "--dart-define=DEV_PASSWORD=SegmentsDevQA2024!"
```

Wait for both to compile. Verify with:
```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:3005
curl -s -o /dev/null -w "%{http_code}" http://localhost:3006
```

If either app shows onboarding/login, you'll need to click through it once using Playwright before verification agents can test screens.

### 0.4 Login Both Apps via Playwright

Navigate Playwright to each port. If login screen appears:
1. Look for "Dev login" or similar dev shortcut
2. Enter password: `SegmentsDevQA2024!`
3. Click through any onboarding steps
4. Verify home screen loads

---

## Phase 1 -- Widget Conversion (Batch)

Convert ALL unconverted widgets BEFORE any screens. This eliminates dependency issues.

### Widget Priority Order
Convert in this order (most depended-on first):
1. `bottom_nav_scaffold.dart` -- used by almost every screen
2. `shimmer_loading.dart` -- used in loading states
3. `user_avatar.dart` -- used in profiles, nav
4. `accent_headline.dart` -- used in section headers
5. `morning_checkin_card.dart` -- home screen
6. `today_workout_card.dart` -- home screen
7. `activity_charts.dart`, `activity_analysis.dart`, `activity_laps_table.dart`, `activity_splits_table.dart`, `activity_map.dart` -- activity detail
8. `best_efforts_list.dart`, `segment_efforts_list.dart` -- activity detail
9. `chat_message_widget.dart` -- AI coach
10. `training_plan_preview.dart`, `phase_timeline.dart`, `week_card.dart`, `week_edit_sheet.dart`, `weekly_progress_bar.dart` -- plans
11. `knowledge_article_card.dart`, `knowledge_category_card.dart`, `knowledge_chat_sheet.dart` -- knowledge
12. `health_status_card.dart`, `readiness_score_badge.dart` -- health
13. `race_countdown_widget.dart` -- races
14. `difficulty_badge.dart`, `key_takeaways_box.dart` -- misc
15. `navigation_drawer.dart` -- side menu
16. All remaining widgets

### Batch Dispatch Strategy

Spawn 2-3 widget conversion agents IN PARALLEL (they edit different files, no conflict):

```
Agent tool:
  description: "Convert widget [name]"
  subagent_type: general-purpose
  run_in_background: true
  prompt: [WIDGET CONVERSION PROMPT -- see template below]
```

After each batch completes, run:
```bash
cd "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend" && flutter analyze 2>&1 | tail -30
```

If errors exist, spawn fix agents. Update REDESIGN_TODO.md after each batch.

### Widget Conversion Agent Prompt Template

```
You are a Vettra Widget Conversion Agent. Convert ONE widget to the Noir design system.

PROJECT: /Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend
WIDGET TO CONVERT: lib/widgets/[WIDGET_NAME].dart
OLD APP REFERENCE: /Users/jakobwredstrom/Desktop/Vetra App/vettra-app/lib/widgets/[WIDGET_NAME].dart

[PASTE THE ENTIRE DESIGN SYSTEM REFERENCE BLOCK FROM ABOVE]

STEPS:
1. Read the BRAND GUIDE first: /Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/BRAND.md
2. Read the widget file in the NEW app: lib/widgets/[WIDGET_NAME].dart
2. Read the SAME widget in the OLD app (if it exists) for functional reference
3. Convert ALL colors to NoirColors.of(context) tokens
4. Convert ALL text styles to NoirTypography.xxx
5. Remove ALL shadows -- replace with Border.all(color: c.border)
6. Remove ALL hardcoded colors (Colors.xxx, Color(0xFFxxxxxx))
7. Remove import '../utils/theme.dart' -- add imports for colors.dart and typography.dart
8. Preserve ALL functionality -- every callback, every parameter, every layout
9. Run: cd "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend" && flutter analyze lib/widgets/[WIDGET_NAME].dart 2>&1 | tail -20
10. If 0 errors: git add lib/widgets/[WIDGET_NAME].dart && git commit -m "design: convert [WIDGET_NAME] to Noir"
11. Report: "CONVERTED: [WIDGET_NAME]" or "FAILED: [reason]"

RULES:
- NO emojis in UI text
- NO MockData
- Preserve every parameter, callback, and layout structure
- Only change colors, text styles, shadows, and imports
- If a color is semantic (green=good, red=bad), map it to the nearest Noir token:
  green -> NoirColors.hrvColor (#4A8C5C)
  red -> NoirColors.crimson (#B5363B)
  gray -> c.textMuted
  blue -> c.accent (or NoirColors.crimson for CTAs)
```

---

## Phase 2 -- Screen Conversion (Sequential, One at a Time)

### Screen Priority Order

Convert in this order (most visible/important first):

**Tier 1 -- Core screens (convert first):**
1. feed_screen.dart
2. training_plans_screen.dart
3. ai_trainer_screen.dart
4. profile_screen.dart
5. settings_screen.dart

**Tier 2 -- Important feature screens:**
6. health_screen.dart
7. health_insights_screen.dart
8. fitness_dashboard_screen.dart
9. knowledge_hub_screen.dart
10. knowledge_article_screen.dart
11. races_screen.dart
12. race_detail_screen.dart
13. race_edit_screen.dart
14. personal_info_screen.dart
15. personal_records_screen.dart
16. running_profile_screen.dart
17. morning_checkin_screen.dart

**Tier 3 -- Secondary screens:**
18. workout_builder_screen.dart
19. workout_execution_screen.dart
20. workout_library_screen.dart
21. workout_summary_screen.dart
22. record_workout_screen.dart
23. active_workout_screen.dart
24. plan_wizard_screen.dart
25. plan_generation_screen.dart
26. plan_reveal_screen.dart
27. coach_evaluation_wizard_screen.dart
28. intelligence_reveal_screen.dart
29. squad_screen.dart
30. squad_detail_screen.dart
31. squad_invite_screen.dart

**Tier 4 -- Low-priority screens:**
32. injury_log_screen.dart
33. connections_screen.dart
34. connect_device_screen.dart
35. developer_connections_screen.dart
36. deals_screen.dart
37. gear_deals_screen.dart
38. shoe_deals_screen.dart
39. data_processing_screen.dart
40. notification_settings_screen.dart
41. legal_screen.dart
42. paywall_screen.dart
43. audio_settings_screen.dart
44. whoop_screen.dart
45. strength_glossary_screen.dart

**Tier 5 -- Auth/Onboarding (convert last -- fragile):**
46. auth_screen.dart
47. onboarding_screen.dart
48. onboarding_splash_screen.dart
49. onboarding_complete_screen.dart
50. performance_screen.dart

Skip: home_feed.dart, activities_screen.dart, activity_detail_screen.dart, ai_coach_screen.dart, plans_screen.dart, app_shell.dart (already converted or Noir-native)

### Per-Screen Conversion Loop

For each unconverted screen:

**Step A -- Dispatch Conversion Agent**

```
Agent tool:
  description: "Convert [screen_name]"
  subagent_type: general-purpose
  run_in_background: false   <-- WAIT for result
  prompt: |
    You are a Vettra Screen Conversion Agent. Convert one screen to the Noir aesthetic.

    PROJECT: /Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend
    SCREEN: lib/screens/[SCREEN_NAME].dart
    OLD APP REFERENCE: /Users/jakobwredstrom/Desktop/Vetra App/vettra-app/lib/screens/[SCREEN_NAME].dart

    [PASTE THE ENTIRE DESIGN SYSTEM REFERENCE BLOCK]

    YOUR PROCESS:
    1. Read the BRAND GUIDE: /Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend/BRAND.md -- this is the authoritative design reference with all color tokens, typography rules, component patterns, and absolute rules
    2. Read the OLD screen file (in vettra-app) -- this is the source of truth for functionality
       Note every: provider (ref.watch/ref.read), navigation (Navigator.push), tap handler, API call, data model
    3. Read the NEW screen file (in vettra-newfrontend) -- this is what you're converting
    4. IMPORTANT: Before you start converting, invoke the frontend-design skill (use the Skill tool with skill: "frontend-design") to get aesthetic guidance for this specific screen. Describe the screen's purpose and content, and ask how to apply the Noir design system with taste and intention. Follow its guidance for spatial composition, typography hierarchy, and color usage.
    4. Rewrite the screen:
       - Replace ALL AppTheme.xxx and hardcoded colors with NoirColors tokens
       - Replace ALL inline TextStyle/GoogleFonts with NoirTypography.xxx
       - Remove ALL BoxShadow -- use Border.all(color: c.border)
       - Remove import '../utils/theme.dart' -- add theme/colors.dart + theme/typography.dart
       - Apply the frontend-design guidance for layout, spacing, visual hierarchy
       - KEEP every provider, every navigation, every tap handler, every API call IDENTICAL
    5. Run: cd "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend" && flutter analyze lib/screens/[SCREEN_NAME].dart 2>&1 | tail -20
    6. Fix any analysis errors
    7. Run flutter analyze again -- must be 0 errors
    8. git add lib/screens/[SCREEN_NAME].dart && git commit -m "design: convert [SCREEN_NAME] to Noir aesthetic"
    9. Report: "CONVERTED: [SCREEN_NAME]" with a 2-sentence summary of what changed
       OR: "FAILED: [reason]" if you couldn't complete the conversion

    ABSOLUTE RULES:
    1. Every tap, button, navigation from the old screen MUST work identically
    2. Use the SAME Riverpod providers as the old screen
    3. NO MockData -- real providers with AsyncValue.when(loading:, error:, data:)
    4. NO shadows -- 6% opacity borders only
    5. NO hardcoded colors
    6. NO emojis in UI text
    7. Replace any "STRAVA" with "VETTRA"
    8. Maintain ConsumerStatefulWidget / ConsumerWidget pattern
    9. If the screen imports widgets from lib/widgets/, those should already be converted to Noir (they were done in Phase 1). Use NoirColors/NoirTypography in any widget references.
    10. Common pattern: `final c = NoirColors.of(context);` at top of build method
```

**Step B -- Process Result**

If agent reports CONVERTED:
1. Quick verify: `grep -c "NoirColors.of" [file]` should return > 0
2. Quick verify: `grep -c "AppTheme\." [file]` should return 0
3. Update REDESIGN_TODO.md: mark screen as `C` (Converted)

If agent reports FAILED:
1. Log the failure reason in REDESIGN_TODO.md Issues Log
2. Spawn a fix agent with the specific error
3. If fix agent also fails: mark as `NEEDS_MANUAL` and move on (max 2 retries per screen)

**Step C -- Dispatch Verification Agent**

After successful conversion, spawn a verification agent:

```
Agent tool:
  description: "Verify [screen_name]"
  subagent_type: general-purpose
  run_in_background: true   <-- Run in background, continue to next screen
  prompt: |
    You are a Vettra Verification Agent. Test a converted screen using Playwright.

    WHAT TO TEST: [SCREEN_NAME] on http://localhost:3005
    COMPARE WITH: Same screen on http://localhost:3006 (old app)

    Flutter web renders via CanvasKit (canvas). Playwright's accessibility tree may be empty.
    Use COORDINATE-BASED CLICKS (page.mouse.click(x, y)) for Flutter web.
    Wait 5-8 seconds after navigation for Flutter to fully render.

    STEPS:
    1. Navigate Playwright to http://localhost:3005
    2. Wait 6 seconds for Flutter to load
    3. Navigate to the [SCREEN_NAME] screen:
       [PROVIDE NAVIGATION INSTRUCTIONS -- e.g., "Click Activities tab at bottom nav, coordinates approximately (x, y)"]
    4. Take a screenshot with browser_take_screenshot
    5. Evaluate the screenshot:
       - Does the screen render without crashing? (not blank/white)
       - Are there obvious old-design remnants (bright blue, Material shadows, wrong fonts)?
       - Is text readable? Are colors consistent with dark/light theme?
       - Does the layout look intentional and premium?
    6. Click 2-3 interactive elements on the screen (cards, buttons) using coordinate-based clicks
    7. After each click, wait 1-2 seconds and take another screenshot
    8. Verify navigation targets loaded correctly
    9. Check browser console for errors: browser_console_messages
    10. Now navigate to http://localhost:3006 (old app), go to the same screen
    11. Take a screenshot for comparison
    12. Verify: does the new screen show the same data/content as the old?
    13. Report:
        "VERIFIED: [SCREEN_NAME] -- renders correctly, N/N interactive elements work, matches old app"
        OR "ISSUES_FOUND: [SCREEN_NAME] -- [list each issue]"

    LOGIN: If app shows login screen:
    - Look for "Dev login" button
    - Enter password: SegmentsDevQA2024!
    - Click through any onboarding

    RULES:
    - Never ask questions
    - Take at most 5 screenshots total (to conserve context)
    - Focus on: does it render? does it crash? do clicks work? does data load?
```

**Step D -- Process Verification Result**

When verification agent reports back (may be background):
- VERIFIED: Update REDESIGN_TODO.md, mark screen as `V`
- ISSUES_FOUND: Spawn fix agents for each issue, then re-verify

**Step E -- Checkpoint**

After every 3 screens:
1. Update REDESIGN_TODO.md with all results
2. Run: `cd "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend" && flutter analyze 2>&1 | tail -20`
3. If errors: spawn fix agent
4. Commit: `git add REDESIGN_TODO.md && git commit -m "chore: redesign checkpoint -- N/M screens done"`
5. Append to Progress Log in REDESIGN_TODO.md

After 8-10 screens: SERIOUSLY consider stopping (update NEXT ACTION and let next session continue). Context health > completing one more screen.

---

## Phase 3 -- Playwright Deep Verification (Full Walkthrough)

After ALL screens are converted, do a comprehensive walkthrough. Spawn ONE large verification agent:

```
Agent tool:
  description: "Full app walkthrough"
  subagent_type: general-purpose
  run_in_background: false
  prompt: |
    You are a Vettra Full Walkthrough Agent. Test the entire app using Playwright.

    NEW APP: http://localhost:3005
    OLD APP: http://localhost:3006

    Flutter web uses CanvasKit -- use coordinate-based clicks, wait 5-8s after navigation.

    TEST EVERY TAB:
    1. Home tab -- verify cards load, content visible
    2. Activities tab -- verify list loads, tap first activity -> detail screen loads
    3. RUN button (center) -- verify it opens workout/run screen
    4. Plans tab -- verify plan loads or empty state
    5. AI Coach tab -- verify chat interface loads

    TEST KEY USER FLOWS:
    1. Home -> tap activity card -> activity detail (charts, stats, map visible)
    2. Home -> morning check-in card interaction
    3. Activities -> tap activity -> detail -> back -> still on activities
    4. Plans -> tap plan -> plan detail
    5. Profile (via nav drawer or settings icon) -> settings load
    6. Knowledge hub (if accessible) -> tap article -> article renders

    FOR EACH FLOW:
    - Screenshot the key screens (max 15 screenshots total)
    - Do the SAME flow in old app (port 3006) -- verify same behavior
    - Log any differences

    Invoke the frontend-design skill to evaluate 3-4 key screenshots (home, activity detail, plans, AI coach) for aesthetic quality.

    REPORT FORMAT:
    WALKTHROUGH COMPLETE
    Screens tested: N
    Flows tested: N
    Issues found: N
    [List each issue with screen name and description]
    Aesthetic evaluation: [summary from frontend-design]
```

---

## Phase 4 -- QA Marathon

When walkthrough passes, invoke the QA Marathon skill:

```
Skill: qa-marathon
```

This runs the full autonomous QA session. Let it complete fully.

---

## Phase 5 -- Final Click-Through + Report

### 5.1 Final Verification

Spawn one last walkthrough agent (same as Phase 3) to confirm everything still works after QA fixes.

### 5.2 Final Report

Write `REDESIGN_COMPLETE.md` in the project root:

```markdown
# Vettra Redesign -- Completion Report

**Completed**: YYYY-MM-DD HH:MM
**Total screens**: N (M converted, K already Noir, J skipped/manual)
**Total widgets**: N (M converted, K already Noir)
**Issues found and fixed**: N
**Remaining issues**: N

## Screen Status
[Full table from REDESIGN_TODO.md]

## Aesthetic Evaluation
[Summary from frontend-design evaluations]

## Known Limitations
[Any screens that need manual attention]
```

Commit and update REDESIGN_TODO.md with Phase 5 complete.

---

## Fix Agent Template

Use this whenever a bug is found (by any phase):

```
Agent tool:
  description: "Fix [brief description]"
  subagent_type: general-purpose
  run_in_background: true
  prompt: |
    You are a Fix Agent for the Vettra redesign.
    PROJECT: /Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend

    BUG:
    - File: [path]
    - Description: [what's wrong]
    - Expected: [what should happen]
    - Suggested fix: [specific change]

    STEPS:
    1. Read the file
    2. Make the MINIMAL fix (only change what's broken)
    3. Run: flutter analyze [file] 2>&1 | tail -10
    4. If 0 errors: git add [file] && git commit -m "fix: [description]"
    5. Report: "FIXED: [what changed]" or "FAILED: [reason]"

    RULES: Never ask questions. Minimal changes only. No refactoring.
```

---

## Flutter Web + Playwright Notes

Flutter web renders to a **canvas** (CanvasKit mode):

1. Playwright accessibility tree is usually EMPTY for Flutter -- don't rely on `browser_snapshot`
2. Use **coordinate-based clicks**: `page.mouse.click(x, y)`
3. **Screenshots DO work** -- use `browser_take_screenshot`
4. Wait **5-8 seconds** after navigation for Flutter to render
5. **Hot reload is unreliable** for Flutter web -- restart the Flutter process after code changes:
   ```bash
   # Kill old process
   kill [PID]
   # Start new
   cd "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend" && flutter run -d chrome --web-port=3005 ...
   ```
6. After restarting, wait for compilation + initial render (can take 30-60 seconds)

### Playwright Login Flow
If the app shows onboarding/login:
1. Click through splash pages ("Get started" or "Next")
2. Click "Dev login" at bottom of login screen
3. Enter password: `SegmentsDevQA2024!`
4. Click "Sign in"
5. If onboarding appears, click through all steps (select defaults, click NEXT)

### Navigation Coordinates (approximate, for 1280x800 viewport)
- Home tab: (128, 760)
- Activities tab: (320, 760)
- RUN button: (640, 740)
- Plans tab: (960, 760)
- AI Coach tab: (1152, 760)
- First card on any list: (640, 300)
- Back button (top-left): (40, 40)
- Settings/menu (top-right): (1240, 40)

These are approximate -- verification agents should take a screenshot first and adjust coordinates based on actual layout.

---

## Operating Principles

**You are a dispatcher, not a worker.** Your context is precious. Every file you read, every screenshot you process, every line of code you write is wasted coordinator context. Sub-agents have fresh context -- use it.

**Never stop early.** If there are 50 screens, dispatch 50 conversion agents. If a screen fails, log it and move on.

**Never ask questions.** If ambiguous, make the best decision and log it.

**Spawn agents immediately.** Don't plan 10 screens ahead -- dispatch the next one as soon as the current one finishes.

**Checkpoint religiously.** REDESIGN_TODO.md is your lifeline. Update it after every screen. Commit after every 3 screens.

**Max 2 retries per screen.** If conversion + 2 fix attempts fail, mark NEEDS_MANUAL and move on. Don't waste context on one stubborn screen.

**Widget-first, screen-second.** Phase 1 (widgets) MUST complete before Phase 2 (screens) begins. This eliminates 90% of dependency issues.

**Background verification, foreground conversion.** Conversion agents run in foreground (you need their result). Verification agents run in background (you can start the next conversion while verification happens).

**8-10 screens per session is realistic.** Don't try to heroically fit 50 screens in one session. Context exhaustion after 8-10 is expected and handled by REDESIGN_TODO.md.

---

## Recovery Scenarios

### Flutter process dies
```bash
# Find and kill stale Flutter processes
ps aux | grep flutter | grep -v grep | awk '{print $2}' | xargs kill 2>/dev/null
# Restart
cd "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend" && flutter run -d chrome --web-port=3005 --dart-define=USE_PRODUCTION_URL=true --dart-define=DEV_EMAIL=devqa@segments.app "--dart-define=DEV_PASSWORD=SegmentsDevQA2024!"
```

### Conversion agent breaks the app (flutter analyze has errors)
```bash
# Revert the broken file
cd "/Users/jakobwredstrom/Desktop/Vetra App/vettra-newfrontend" && git checkout -- lib/screens/[broken_file].dart
# Re-run flutter analyze to confirm clean
flutter analyze 2>&1 | tail -10
```

### Sub-agent returns FAILED
1. Read the failure reason (it's in the agent result -- don't read files)
2. Spawn a fix agent with the specific error message
3. If fix agent also fails: mark NEEDS_MANUAL in REDESIGN_TODO.md
4. Move to next screen

### Context getting long
You'll feel it: responses get slower, tool calls take longer. When you notice:
1. STOP converting new screens
2. Process any pending verification agent results
3. Update REDESIGN_TODO.md with exact NEXT ACTION
4. Commit
5. STOP

---

## When to Stop (the session, not the marathon)

**Stop this session when ANY of these are true:**
- You've processed 8-10 screens and context feels heavy
- A tool call times out or returns truncated results
- You notice yourself re-reading REDESIGN_TODO.md multiple times (sign of context pressure)

**The marathon is COMPLETE when ALL of these are true:**
1. Every screen marked C+V in REDESIGN_TODO.md (or NEEDS_MANUAL with explanation)
2. Every widget converted
3. Phase 3 walkthrough passed
4. Phase 4 QA marathon passed
5. Phase 5 final report written
6. `REDESIGN_COMPLETE.md` committed
