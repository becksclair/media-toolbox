# Tasks: Tailwind CSS v4 & shadcn Setup

**Input**: Design documents from `/specs/001-tailwind-shadcn-setup/`
**Prerequisites**: plan.md (required), spec.md (required)

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Extension project**: `toolbox-ext/` (root relative to monorepo)
- **Config files**: `toolbox-ext/wxt.config.ts`, `toolbox-ext/tsconfig.json`, etc.
- **Components**: `toolbox-ext/components/ui/`
- **Entrypoints**: `toolbox-ext/entrypoints/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and configuration setup

- [ ] T001 Install Tailwind v4 dependencies in toolbox-ext: `bun add tailwindcss@next @tailwindcss/vite@next`
- [ ] T002 [P] Update toolbox-ext/wxt.config.ts to add @tailwindcss/vite plugin to Vite config
- [ ] T003 [P] Update toolbox-ext/tsconfig.json to add path alias `"@/*": ["./*"]`
- [ ] T004 Verify build succeeds: `cd toolbox-ext && bun build` with no errors or warnings

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that blocks all user stories

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T005 Initialize shadcn in toolbox-ext: `bunx --bun shadcn@latest init` (interactive, select: New York, Neutral, CSS variables: Yes, CSS file: entrypoints/app.css)
- [ ] T006 [P] Add Button component: `bunx --bun shadcn@latest add button` in toolbox-ext
- [ ] T007 [P] Add Card component: `bunx --bun shadcn@latest add card` in toolbox-ext
- [ ] T008 Verify component files exist: toolbox-ext/components/ui/button.tsx and toolbox-ext/components/ui/card.tsx (check with `ls -la`)

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - UI Styling Foundation Ready (Priority: P1) 🎯 MVP

**Goal**: Tailwind CSS v4 configured and compiling correctly so developers can style components with utility classes

**Independent Test**: Run extension in dev mode (`bun dev`), inspect popup in Chrome DevTools, verify Tailwind classes (e.g., `text-white`, `bg-blue-500`, `p-4`) are applied and CSS file is bundled

### Implementation for User Story 1

- [ ] T009 [US1] Update toolbox-ext/entrypoints/app.css: Verify `@import "tailwindcss"` is present at top, CSS variables defined in :root
- [ ] T010 [US1] Update toolbox-ext/entrypoints/popup/main.tsx: Add `import '../app.css'` at very top (before React imports)
- [ ] T011 [US1] Start dev server: `cd toolbox-ext && bun dev` and verify no CSS compilation errors in console
- [ ] T012 [US1] Verify CSS compilation: Open Chrome DevTools → Network tab → filter for "popup-" → confirm popup-[hash].css file exists with size > 10KB

**Checkpoint**: User Story 1 complete - Tailwind CSS v4 is configured and CSS compiles into the extension build

---

## Phase 4: User Story 2 - Pre-built Component Library Available (Priority: P1)

**Goal**: shadcn Button and Card components available and importable without TypeScript errors

**Independent Test**: Import Button and Card in a TypeScript file, verify no `Cannot find module` errors; check components/ui/ directory contains both files

### Implementation for User Story 2

- [ ] T013 [US2] Verify Button component importable: Check toolbox-ext/components/ui/button.tsx exists and has `export { Button }` statement
- [ ] T014 [US2] Verify Card component importable: Check toolbox-ext/components/ui/card.tsx exists and has `export { Card, CardHeader, CardTitle, CardContent }` statements
- [ ] T015 [US2] Verify utils imported cleanly: Check toolbox-ext/lib/utils.ts exists and exports `cn` function
- [ ] T016 [US2] Test import resolution: Create temp file with `import { Button } from '@/components/ui/button'` and run `bun compile` (TypeScript type check only) - should pass with no errors

**Checkpoint**: User Story 2 complete - shadcn components installed, importable, and typed correctly

---

## Phase 5: User Story 3 - Popup Showcases Working UI (Priority: P1)

**Goal**: Extension popup displays styled Button and Card components demonstrating complete Tailwind + shadcn integration

**Independent Test**: Load extension in Chrome dev mode, open popup, visually confirm Card contains Button with Tailwind styling applied (colors, spacing, hover effects)

### Implementation for User Story 3

- [ ] T017 [US3] Update toolbox-ext/entrypoints/popup/App.tsx: Import Button from '@/components/ui/button' and Card, CardHeader, CardTitle, CardContent from '@/components/ui/card'
- [ ] T018 [US3] Update toolbox-ext/entrypoints/popup/App.tsx: Render Card wrapper with CardHeader containing CardTitle "Video Fetch Extension"
- [ ] T019 [US3] Update toolbox-ext/entrypoints/popup/App.tsx: Render CardContent with descriptive paragraph text and Button with className="w-full" and onClick handler (log or toast)
- [ ] T020 [US3] Apply Tailwind classes: Ensure component uses classes like `p-4`, `space-y-4`, `text-sm`, `text-muted-foreground` in appropriate elements
- [ ] T021 [US3] Start dev server: `cd toolbox-ext && bun dev` and verify popup opens in Chrome without TypeScript errors
- [ ] T022 [US3] Visual verification: Open extension popup in Chrome, inspect that Card is visible with proper spacing, Button is clickable with hover effects, colors match Tailwind design system

**Checkpoint**: User Story 3 complete - Popup displays full-stack working UI with styled components

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Production readiness and verification

- [ ] T023 [P] Verify production build: Run `cd toolbox-ext && bun build` and confirm completes without errors or warnings
- [ ] T024 [P] Inspect bundled CSS: Check `.output/chrome-mv3/assets/popup-[hash].css` contains CSS variable definitions and Button/Card styles (use `cat` or text editor)
- [ ] T025 Load extension in Chrome unpacked: Navigate to `chrome://extensions`, enable Developer mode, click "Load unpacked", select `toolbox-ext/.output/chrome-mv3`
- [ ] T026 Test production popup: Click extension icon, verify popup displays identically to dev mode with full styling
- [ ] T027 [P] Check console for errors: Open Chrome DevTools (popup context), confirm console shows zero errors or warnings
- [ ] T028 Verify TypeScript strict mode: Run `cd toolbox-ext && bun compile` (type check) and confirm zero errors (all components properly typed)
- [ ] T029 Run quickstart validation: `cd toolbox-ext && bun dev` and leave running for 30 seconds, confirm HMR working (no stale state errors)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-5)**: All depend on Foundational phase completion
  - User stories can proceed in parallel after Foundational (or sequentially in priority order)
  - All three user stories are P1 and independent (no cross-dependencies)
- **Polish (Phase 6)**: Depends on all user stories being complete

### Parallel Opportunities

**Phase 1 Setup** (all independent, different files):
- T001: Install deps
- T002: wxt.config.ts update
- T003: tsconfig.json update
Can run T002 and T003 in parallel while T001 installs

**Phase 2 Foundational** (shadcn init blocks component adds):
- T005: shadcn init (must run first)
- T006 + T007: Add Button and Card in parallel (both depend only on T005, different files)
- T008: Verification (depends on T006, T007)

**Phase 3-5 User Stories** (all independent after Foundational):
- Once Phase 2 complete, all three user story phases can run in parallel:
  - Developer A: Phase 3 (US1) - CSS integration
  - Developer B: Phase 4 (US2) - Component verification
  - Developer C: Phase 5 (US3) - Popup UI showcase
- If single developer: Sequential P1 → P2 → P3 (all P1, so order doesn't matter)

**Phase 6 Polish** (mostly independent):
- T023, T024 can run in parallel (different verification tasks)
- T025-T029 are sequential (build → inspect → load → test → verify)

### Within Each User Story

- **User Story 1**: Linear sequence (T009 → T010 → T011 → T012)
- **User Story 2**: Linear sequence (T013 → T014 → T015 → T016)
- **User Story 3**: Linear sequence (T017 → T018 → T019 → T020 → T021 → T022)

---

## Parallel Example: Full Implementation with 3 Developers

```bash
# Terminal 1: Phase 1 Setup (run sequentially, quick)
cd /home/bex/projects/media-toolbox/toolbox-ext
bun add tailwindcss@next @tailwindcss/vite@next  # T001

# Terminal 2: Phase 1 parallel updates
# Edit wxt.config.ts                            # T002 (start while T001 installs)
# Edit tsconfig.json                            # T003 (start while T001 installs)

# Terminal 3: Verify Phase 1
bun build                                       # T004 (after T001 completes)

# Once Phase 1 complete:
bunx --bun shadcn@latest init                  # T005 (Phase 2 foundational, interactive)

# Once T005 complete, run these in parallel (Phase 2):
bunx --bun shadcn@latest add button            # T006
bunx --bun shadcn@latest add card              # T007

# Phase 2 verification:
ls -la components/ui/button.tsx components/ui/card.tsx  # T008

# Once Phase 2 complete, all three user stories in parallel:

# Developer A - Phase 3 (US1):
# Edit entrypoints/app.css (T009)
# Edit entrypoints/popup/main.tsx, add CSS import (T010)
# bun dev (T011)
# Inspect DevTools Network tab (T012)

# Developer B - Phase 4 (US2):
# Verify components/ui/button.tsx (T013)
# Verify components/ui/card.tsx (T014)
# Verify lib/utils.ts (T015)
# bun compile for type check (T016)

# Developer C - Phase 5 (US3):
# Edit entrypoints/popup/App.tsx, add imports (T017)
# Edit App.tsx, add Card render (T018)
# Edit App.tsx, add CardContent (T019)
# Edit App.tsx, apply Tailwind classes (T020)
# bun dev (T021)
# Visual inspection in Chrome popup (T022)

# Phase 6 Polish (sequential, after all stories):
bun build                                      # T023
cat .output/chrome-mv3/assets/popup-*.css     # T024
# Load in Chrome, test (T025-T027)
bun compile                                    # T028
bun dev # verify HMR (T029)
```

---

## Implementation Strategy

### MVP First (Recommended)

1. **Complete Phase 1**: Setup (5-10 minutes)
   - Install deps
   - Update configs
   - Verify build

2. **Complete Phase 2**: Foundational (5-10 minutes)
   - Initialize shadcn
   - Add Button + Card components
   - Verify imports

3. **Complete Phase 3**: User Story 1 (5-10 minutes)
   - Wire CSS into popup
   - Start dev server
   - Verify CSS compiles

4. **STOP and VALIDATE**: User Story 1 independently
   - Dev server running
   - CSS files exist
   - Move to Phase 4 only after successful validation

### Incremental Delivery (with 3 Developers)

1. Developers work together on Phase 1 + Phase 2 (setup infrastructure)
2. Once infrastructure ready:
   - Developer A: Complete US1 (CSS foundation)
   - Developer B: Verify US2 (components available)
   - Developer C: Implement US3 (popup showcase)
3. Merge all three stories
4. Run Phase 6 Polish together

### Sequential (Single Developer)

1. Phase 1 → Phase 2 → Phase 3 → Phase 4 → Phase 5 → Phase 6
2. Total estimated time: 45-60 minutes
3. Each user story independently testable; can deploy after each story if desired

---

## Notes

- [P] tasks = different files, no dependencies on incomplete tasks
- [Story] label maps task to specific user story for traceability
- Each user story independently completable and testable
- All Phases 1-2 complete before any user story begins
- TypeScript strict mode enforced throughout; `bun compile` verifies all types
- HMR (hot module replacement) for CSS and JS during dev; restart if issues persist
- Chrome DevTools required for visual verification during testing
- Commit after each major phase (or after each user story) for clear history

