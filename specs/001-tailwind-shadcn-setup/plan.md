# Implementation Plan: Tailwind CSS v4 & shadcn Setup

**Branch**: `001-tailwind-shadcn-setup` | **Date**: 2025-10-29 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `specs/001-tailwind-shadcn-setup/spec.md`

## Summary

Set up Tailwind CSS v4 and shadcn component library in the WXT-based Chrome extension to enable rapid, consistent UI development. Tailwind v4 provides a modern, configuration-free approach via Vite plugin integration. shadcn provides copy-paste components built on Radix UI primitives (Button, Card) that are fully customizable and production-ready. The feature delivers foundational UI infrastructure enabling the extension popup to use a design system.

**Technical Approach**: Leverage WXT's built-in Vite integration to install Tailwind v4 via `@tailwindcss/vite` plugin, initialize shadcn with `bunx shadcn@latest init`, add Button and Card components, and verify the complete CSS compilation pipeline by updating the popup UI to showcase both components.

## Technical Context

**Language/Version**: TypeScript 5.0+, React 18.2+, Node.js/Bun runtime

**Primary Dependencies**:
- `tailwindcss@next` (v4-beta) - CSS utility framework
- `@tailwindcss/vite@next` - Vite plugin for Tailwind compilation
- `shadcn/ui` (latest) - Component library (copy-paste components)
- `@radix-ui/*` (latest) - Accessible component primitives (auto-installed by shadcn)
- `clsx`, `tailwind-merge` - Utility functions (auto-installed by shadcn)
- `wxt@0.19.0+` - Web extension framework (already configured)
- `bun@1.0.0+` - Package manager and runtime (already configured)

**Storage**: N/A (configuration-only, no persistent data)

**Testing**: Playwright (for extension popup UI testing), Chrome DevTools (visual inspection)

**Target Platform**: Chrome extension (Manifest V3), WXT development environment

**Project Type**: Web extension (single entrypoint focus: popup UI)

**Performance Goals**:
- CSS bundle size: <100 KB (Tailwind optimized)
- Build time: <5 seconds (Vite + Tailwind v4)
- Hot reload latency: <200ms for CSS changes
- Runtime: Popup renders in <500ms (React + styled components)

**Constraints**:
- Must work in isolated popup context (no shadow DOM needed)
- TypeScript strict mode required
- No custom CSS files (Tailwind-only styling)
- Components must be type-safe (no `any` types)

**Scale/Scope**: Single popup entrypoint, 2 initial components (Button, Card), extensible for future UI features

## Constitution Check

**Media Toolbox Constitution v1.0.0 Alignment**:

| Principle | Alignment | Notes |
|-----------|-----------|-------|
| **I. Modular Separation** | ✅ Pass | shadcn components are independent, copy-paste modules; Tailwind config isolated to extension |
| **II. Type Safety & Correctness** | ✅ Pass | TypeScript strict mode enforced; shadcn components fully typed; path aliases configured in tsconfig.json |
| **III. Bounded Resources** | ✅ Pass | CSS bundle size bounded by Tailwind tree-shaking; popup payload constrained by extension context |
| **IV. Observable Operations** | ✅ Pass | Build logs show CSS compilation; DevTools/console visible for styling issues |
| **V. Progressive Delivery** | ✅ Pass | Feature delivered as independent MVP: Button + Card showcase components; can be extended incrementally with additional shadcn components |

**No Constitution violations. Feature aligns with all 5 core principles.**

## Project Structure

### Documentation (this feature)

```text
specs/001-tailwind-shadcn-setup/
├── spec.md                    # Feature specification (user stories, requirements)
├── plan.md                    # This file (implementation approach)
├── research.md                # Technical research on WXT, Tailwind v4, shadcn
├── data-model.md              # N/A (configuration-only, no data model)
├── quickstart.md              # N/A (see plan.md for getting started)
├── contracts/                 # N/A (no API contracts for UI infrastructure)
└── checklists/
    └── requirements.md        # Quality validation checklist
```

### Source Code (toolbox-ext component)

```text
toolbox-ext/
├── wxt.config.ts              # [MODIFY] Add @tailwindcss/vite plugin
├── tsconfig.json              # [MODIFY] Add path alias @/* -> ./*
├── package.json               # [MODIFY] Add dependencies
├── components.json            # [CREATE] shadcn configuration
├── entrypoints/
│   ├── app.css                # [CREATE] Tailwind v4 entry point with CSS variables
│   └── popup/
│       ├── main.tsx           # [MODIFY] Import app.css before React.render()
│       └── App.tsx            # [MODIFY] Use Button + Card components
├── components/
│   ├── ui/
│   │   ├── button.tsx         # [CREATE] shadcn Button component
│   │   └── card.tsx           # [CREATE] shadcn Card component
│   └── ...                    # Future components
└── lib/
    └── utils.ts               # [CREATE] shadcn cn() utility
```

**Structure Decision**: Single extension project with isolated popup UI. Components directory follows shadcn convention (`components/ui/`). Styles centralized in `entrypoints/app.css`. Configuration files support TypeScript path aliases for clean imports.

## Phase Breakdown

### Phase 0: Setup (Infrastructure)

**Objective**: Install dependencies and configure build pipeline for Tailwind + shadcn

**Tasks**:
1. Install Tailwind v4 and Vite plugin: `bun add tailwindcss@next @tailwindcss/vite@next`
2. Update `wxt.config.ts` to add `@tailwindcss/vite` plugin
3. Update `tsconfig.json` to add `@/*` path alias for component imports
4. Initialize shadcn: `bunx --bun shadcn@latest init` (generates `components.json`, `lib/utils.ts`, `entrypoints/app.css`)
5. Verify build succeeds: `bun build`

**Verification**:
- `bun build` completes without errors
- `entrypoints/app.css` contains Tailwind directives + CSS variables
- `components.json` configured with correct paths
- `tsconfig.json` has `"@/*": ["./*"]` paths

### Phase 1: Component Library (shadcn)

**Objective**: Install Button and Card components, making them available for use

**Tasks**:
1. Add Button component: `bunx --bun shadcn@latest add button`
2. Add Card component: `bunx --bun shadcn@latest add card`
3. Verify component files created: `components/ui/button.tsx`, `components/ui/card.tsx`
4. Verify TypeScript paths resolve correctly (no import errors)

**Verification**:
- Files exist: `components/ui/button.tsx`, `components/ui/card.tsx`
- `package.json` includes peer dependencies (`@radix-ui/react-*`)
- No TypeScript errors on import: `import { Button } from '@/components/ui/button'`

### Phase 2: Popup Integration (CSS Import)

**Objective**: Wire CSS into popup entrypoint so styles are available at runtime

**Tasks**:
1. Update `entrypoints/popup/main.tsx`: Add `import './app.css'` at top (before ReactDOM.render)
2. Verify styles load in dev mode: `bun dev` then inspect popup in Chrome

**Verification**:
- Dev server starts without CSS errors
- Chrome DevTools shows `popup-[hash].css` in Network tab
- Inline `<style>` tags visible in popup HTML (WXT injects compiled CSS)

### Phase 3: Popup UI Showcase (Components)

**Objective**: Update popup App.tsx to use Button and Card components, demonstrating the full stack

**Tasks**:
1. Update `entrypoints/popup/App.tsx`:
   - Import Button from `@/components/ui/button`
   - Import Card from `@/components/ui/card`
   - Render a Card containing descriptive text and a Button
   - Use Tailwind classes (e.g., `className="p-4"`, `className="text-lg"`)
2. Test in dev mode: Visual inspection confirms styling

**Component Code Example**:
```typescript
import { Button } from '@/components/ui/button';
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card';

export default function App() {
  return (
    <div className="w-80">
      <Card>
        <CardHeader>
          <CardTitle>Video Fetch Extension</CardTitle>
        </CardHeader>
        <CardContent className="space-y-4">
          <p className="text-sm text-muted-foreground">
            Tailwind CSS v4 + shadcn components working correctly.
          </p>
          <Button className="w-full">Download Current Video</Button>
        </CardContent>
      </Card>
    </div>
  );
}
```

**Verification**:
- No TypeScript errors
- No console errors when popup loads
- Card visible with proper spacing and styling
- Button clickable and responsive to hover/active states
- Classes like `w-full`, `space-y-4`, `text-sm` applied correctly

### Phase 4: Production Build (Verification)

**Objective**: Verify feature works identically in production build

**Tasks**:
1. Build for production: `bun build`
2. Check bundled CSS: `cat .output/chrome-mv3/assets/popup-*.css | head -100`
3. Load extension in Chrome (unpacked mode)
4. Open popup and visually verify Button and Card display correctly

**Verification**:
- Production build completes without warnings
- CSS file includes Tailwind utilities and shadcn styles
- Popup renders with identical styling to dev mode
- No missing assets or broken imports in Chrome DevTools

## Implementation Approach

### Technical Decisions

| Decision | Rationale | Alternatives Considered |
|----------|-----------|------------------------|
| **Tailwind v4 (beta)** | Simpler config (no tailwind.config.js), faster builds, modern CSS syntax | Tailwind v3 (stable, but more setup required) |
| **@tailwindcss/vite plugin** | Native Vite integration, WXT-compatible, handles CSS bundling | PostCSS + Tailwind (more complex setup) |
| **shadcn (copy-paste)** | Full customization, no runtime deps, Accessible primitives | Radix UI directly (more configuration), custom components (more time) |
| **CSS variables for theming** | Dynamic theme switching without JS, CSS-standard approach | Inline styles or CSS-in-JS (more code, slower runtime) |
| **Single entrypoint focus** | MVP scope, popup is primary UI surface, reduces complexity | Add content script UI (out of scope, use shadow DOM if needed later) |

### File Modifications Summary

| File | Action | Purpose |
|------|--------|---------|
| `wxt.config.ts` | Add plugin | Register Tailwind Vite plugin |
| `tsconfig.json` | Add paths | Enable `@/` imports for components |
| `package.json` | Add deps | Install Tailwind, shadcn peer deps |
| `entrypoints/app.css` | Create | Tailwind entry point + CSS variables |
| `entrypoints/popup/main.tsx` | Modify | Import CSS before render |
| `entrypoints/popup/App.tsx` | Modify | Use Button + Card components |
| `components/ui/button.tsx` | Create | shadcn Button (generated) |
| `components/ui/card.tsx` | Create | shadcn Card (generated) |
| `lib/utils.ts` | Create | shadcn cn() utility (generated) |
| `components.json` | Create | shadcn config (generated) |

### Risk Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|-----------|
| **Tailwind v4 breaking changes** | Medium | High | Pin exact version in package.json; monitor Tailwind changelog; v4 stable release imminent |
| **CSS not loading in popup** | Low | High | Import CSS in main.tsx BEFORE ReactDOM.render(); verify wxt.config.ts has plugin |
| **Path alias not resolving** | Medium | High | Verify tsconfig.json paths match components.json aliases; clear node_modules if needed |
| **Radix UI peer dep conflicts** | Low | Medium | shadcn handles peer dep installation; manually install if warnings appear |
| **Hot reload CSS issues** | Low | Medium | Restart dev server if styles not updating; check for CSS syntax errors |
| **Content script CSS leakage** | N/A | N/A | Out of scope (popup only); use shadow DOM if content script UI added later |

## Success Criteria for Implementation

| Criterion | Verification | Status |
|-----------|--------------|--------|
| **Build succeeds** | `bun build` completes without errors | Phase 0 gate |
| **CSS compiles** | `entrypoints/app.css` generates `popup-[hash].css` | Phase 2 gate |
| **Components import cleanly** | No TypeScript errors for `import { Button }` statements | Phase 1 gate |
| **Popup displays styled** | Visual inspection in Chrome shows Button + Card with correct styling | Phase 3 gate |
| **Dev/prod parity** | Production build renders identically to dev mode | Phase 4 gate |
| **No console errors** | Chrome DevTools console shows no errors when popup loads | Final gate |
| **Hot reload works** | CSS changes reflect immediately in dev mode | Development gate |

## Known Limitations & Future Work

### Out of Scope (Phase 1)

- **Content script UI**: If needed, use shadow DOM with separate CSS bundle
- **Dark mode**: CSS variables support dark mode; toggle can be added later
- **Additional components**: Dialog, Toast, Select, etc. can be added with `bunx shadcn@latest add <component>`
- **Theme customization**: Advanced theming (color schemes, spacing scales) in `entrypoints/app.css`
- **Component customization**: Advanced modifications to Button/Card internals (simple customizations via className props)

### Phase 2+ Opportunities

1. **Extend component library**: Add Dialog, Toast, Dropdown Menu components as features require
2. **Dark mode toggle**: Implement theme switcher using CSS variables
3. **Custom theme**: Define organization-specific colors in app.css `@layer` directives
4. **Responsive UI**: Leverage Tailwind responsive classes (md:, lg:) for future layouts
5. **Accessibility audit**: Test with axe-DevTools, ensure ARIA attributes present

## References

**Official Documentation**:
- WXT Configuration: https://wxt.dev/guide/essentials/config.html
- Tailwind CSS v4: https://tailwindcss.com/blog/tailwindcss-v4-beta
- shadcn/ui Installation: https://ui.shadcn.com/docs/installation/vite
- Radix UI: https://www.radix-ui.com/primitives/docs/overview/introduction

**Tech Stack**:
- TypeScript 5.0+ (strict mode required)
- React 18.2+ (hooks, suspense)
- Vite (module bundling, HMR)
- Bun (package manager, runtime)
- WXT 0.19.0+ (extension framework)

## Assumptions & Dependencies

### Assumptions

1. **WXT project already initialized** with React template
2. **TypeScript strict mode** is enabled (will be by default in new WXT projects)
3. **Bun** is the package manager (already configured in media-toolbox)
4. **Chrome** is the target browser (WXT supports multi-browser, but testing focus is Chrome)
5. **Popup is the primary UI surface** (content scripts, if UI needed, handled separately with shadow DOM)

### External Dependencies

- **Chrome DevTools** for visual verification during development
- **Remote unpacked extension** loading for testing (Chrome Developer Mode required)
- **Internet connection** for fetching component source code during `shadcn add` commands

