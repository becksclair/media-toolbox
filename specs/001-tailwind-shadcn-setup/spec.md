# Feature Specification: Tailwind CSS v4 & shadcn Setup in Chrome Extension

**Feature Branch**: `001-tailwind-shadcn-setup`
**Created**: 2025-10-29
**Status**: Draft
**Input**: User description: "Setup in the Chrome Extension Tailwind CSS v4 and shadcn for ui components, add the packages and setup all the configuration, add the `Button` and `Card` components, and update the extension popup showcasing those components so we know it all works correctly."

## User Scenarios & Testing *(mandatory)*

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE - meaning if you implement just ONE of them,
  you should still have a viable MVP (Minimum Viable Product) that delivers value.
  
  Assign priorities (P1, P2, P3, etc.) to each story, where P1 is the most critical.
  Think of each story as a standalone slice of functionality that can be:
  - Developed independently
  - Tested independently
  - Deployed independently
  - Demonstrated to users independently
-->

### User Story 1 - UI Styling Foundation Ready (Priority: P1)

As an extension developer, I need Tailwind CSS v4 properly configured so that all UI components can be styled consistently and rapidly without writing custom CSS.

**Why this priority**: This is foundational infrastructure. Without it, building the extension UI is slow and error-prone. No other UI work can proceed without this.

**Independent Test**: Can be fully tested by running the extension in dev mode, inspecting the popup to verify Tailwind classes are applied correctly, and confirming the build process includes compiled CSS without errors.

**Acceptance Scenarios**:

1. **Given** the extension is built, **When** the popup is displayed, **Then** Tailwind utility classes (like `text-white`, `bg-blue-500`, `p-4`) are applied without requiring manual CSS files
2. **Given** a developer adds a new component, **When** they use Tailwind class names, **Then** the classes compile and render correctly in the built extension
3. **Given** the extension is deployed, **When** the popup loads, **Then** styles are present (not missing or broken) and match design intent

---

### User Story 2 - Pre-built Component Library Available (Priority: P1)

As an extension developer, I need shadcn components (Button, Card) available and working so I can build the popup UI quickly with accessible, consistent, production-ready components.

**Why this priority**: Shadcn components save significant dev time and ensure accessibility standards are met. Required before implementing the actual popup feature work.

**Independent Test**: Can be fully tested by adding Button and Card components to the popup, verifying they render visually correctly, respond to interactions, and display in the extension UI without console errors.

**Acceptance Scenarios**:

1. **Given** the Button component is installed, **When** it's used in the popup template, **Then** it renders as a clickable button with correct styling
2. **Given** the Card component is installed, **When** it's used in the popup template, **Then** it displays as a styled container with proper spacing and visual hierarchy
3. **Given** both components are in the popup, **When** the extension loads, **Then** no TypeScript errors, console warnings, or styling issues appear

---

### User Story 3 - Popup Showcases Working UI (Priority: P1)

As a developer testing the extension setup, I need the popup to visually demonstrate both Button and Card components so I can confirm the entire UI pipeline (Tailwind + shadcn + build) is functioning correctly.

**Why this priority**: Verification step. Proves all infrastructure is wired correctly end-to-end. Required before moving to real feature development.

**Independent Test**: Can be fully tested by loading the extension in Chrome, opening the popup, and visually confirming that both Button and Card components display with correct Tailwind styling.

**Acceptance Scenarios**:

1. **Given** the extension is loaded in Chrome dev mode, **When** the popup is opened, **Then** a styled Card is visible containing one or more styled Buttons
2. **Given** a Button in the popup is clicked, **When** the click occurs, **Then** the button responds (state change, log entry, or visual feedback)
3. **Given** the extension is built for production, **When** the popup is opened, **Then** styling and components are identical to dev mode

### Edge Cases

- What happens when a developer updates Tailwind config after build? (CSS must recompile on next build)
- How does system handle missing shadcn components? (Build fails with clear error message)
- What if Tailwind and shadcn versions conflict? (Dependencies must be pinned; incompatibilities caught at install time)

## Requirements *(mandatory)*

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right functional requirements.
-->

### Functional Requirements

- **FR-001**: System MUST install Tailwind CSS v4 as a dev dependency in the extension project
- **FR-002**: System MUST install shadcn component library with TypeScript support
- **FR-003**: System MUST compile Tailwind styles into the extension build output so CSS is available at runtime
- **FR-004**: System MUST provide Button and Card components from shadcn that are immediately usable in React/TypeScript code
- **FR-005**: System MUST update the extension popup entrypoint to showcase Button and Card components with no errors
- **FR-006**: System MUST configure TypeScript to properly type shadcn components (no `any` types)
- **FR-007**: System MUST ensure Tailwind configuration respects WXT build conventions (extension bundle, manifest, etc.)

### Key Entities

- **Tailwind CSS Config**: Configuration file defining design tokens, customizations, and plugin settings
- **shadcn Component Registry**: Local shadcn component library (typically in `components/ui/`) containing reusable component definitions
- **Extension Popup**: React component (`entrypoints/popup/App.tsx`) that serves as the visible popup UI in the extension

## Success Criteria *(mandatory)*

<!--
  ACTION REQUIRED: Define measurable success criteria.
  These must be technology-agnostic and measurable.
-->

### Measurable Outcomes

- **SC-001**: Extension builds successfully without TypeScript errors, warnings, or CSS compilation errors
- **SC-002**: Popup displays visually with Button and Card components styled correctly (matches Tailwind design system)
- **SC-003**: Developer can add new Tailwind classes and shadcn components to the popup without modifying build config or adding custom CSS
- **SC-004**: All styling changes are reflected immediately in dev mode (hot reload works correctly)
- **SC-005**: Extension works identically in dev and production builds (no styling regressions when bundled)
