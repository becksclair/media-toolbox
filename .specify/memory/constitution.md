<!-- SYNC IMPACT REPORT
Version Change: None → 1.0.0 (NEW CONSTITUTION)
New Principles: 5 core principles established
Added Sections: Governance, Development Workflow
Ratification Date: 2025-10-29
Modified Templates: spec-template.md, plan-template.md, tasks-template.md reference constitution but do not require updates (structure already compatible)
Follow-up: None
-->

# Media Toolbox Constitution

## Core Principles

### I. Modular Separation

Each component (videofetch backend, toolbox-ext extension, web PWA) MUST be independently versioned, tested, and deployable. Define clear API contracts between components. Use Git submodules for independent repositories. Changes to one component MUST not require coordinated releases of others.

**Rationale**: Media Toolbox spans multiple platforms and technologies. Modularity enables parallel team work, independent release cycles, and reduces integration risk.

### II. Type Safety & Correctness

All new code MUST leverage the host language's type system. Go code MUST use interfaces and proper error handling with stable error strings. TypeScript code MUST have strict mode enabled. Never disable linters or type checkers.

**Rationale**: The architecture spans Go (backend), TypeScript (extension), and future Vite+React (PWA). Type safety catches integration errors early, reduces debugging time, and enables confident refactoring.

### III. Bounded Resources

All components MUST respect resource limits: worker pools (configurable concurrency), bounded queues with backpressure, graceful shutdown handling, rate limiting for APIs. Document all limits in code and configuration.

**Rationale**: videofetch downloads videos concurrently and queues requests. Without bounded resources, the system becomes unpredictable under load and difficult to operate in resource-constrained environments.

### IV. Observable Operations

All components MUST emit structured logs (JSON format preferred) with event types for filtering. Log download lifecycle (start, progress, complete, error), API requests, system events. Enable runtime debugging via log levels and filtering without code changes.

**Rationale**: Media Toolbox runs long-lived download operations. Users and operators need visibility into system state and failure modes without rebuilding or adding debug code.

### V. Progressive Delivery

Implement features as independent, testable user stories. Each story MUST deliver measurable value independently. Target MVP-first: ship working feature → gather feedback → iterate. Accept technical debt in scaffold phase (extension, PWA); production components (videofetch backend) require higher quality.

**Rationale**: The project spans multiple components at different maturity levels. Progressive delivery allows the team to ship early wins (backend is production-ready, extension is MVP), gather user feedback, and prioritize next work.

## Development Workflow

### Code Review & Quality Gates

1. **Type Check**: All languages MUST pass type checking (tsc, golangci-lint)
2. **Lint**: oxlint for TypeScript, golangci-lint for Go; no disable directives
3. **Format**: oxfmt (TypeScript), gofmt (Go); enforce in CI
4. **Test**: Unit tests required for core logic; integration tests for cross-component flows
5. **Build**: Project MUST build successfully before merge

### Component-Specific Expectations

**videofetch (Backend - Production Grade)**:
- All API handlers MUST have tests
- Database operations MUST use prepared statements
- Download manager MUST handle graceful shutdown
- yt-dlp integration failures MUST be logged with stable error strings

**toolbox-ext (Extension - MVP Stage)**:
- Scaffold only; feature tests added when moving to beta
- WXT conventions MUST be followed (entrypoint auto-discovery)
- Manifest V3 compliance required

**web (PWA - Planned)**:
- Not yet implemented; future constitution amendments as scope clarifies

### Testing Strategy

| Component      | Unit Tests | Integration Tests | Notes                                    |
|----------------|------------|-------------------|------------------------------------------|
| videofetch     | MUST       | MUST              | Race detector enabled; tags=integration  |
| toolbox-ext    | Optional   | Optional          | Planned when feature-complete            |
| web            | TBD        | TBD               | Scope pending                            |

### Graceful Degradation

When one component is unavailable, others MUST continue functioning. Example: Extension should queue URLs locally if backend is down; backend should serve cached metadata if yt-dlp fails temporarily.

## Governance

### Amendment Process

1. Changes to principles require documented rationale and impact analysis
2. All amendments update the version number (semantic versioning)
3. Version changes TRIGGER cascading updates to spec-template.md, plan-template.md, tasks-template.md
4. Amendments are effective immediately on commit to main

### Compliance Verification

- All pull requests MUST reference this constitution in review comments if architecture/process changes occur
- During feature planning (spec.md), include "Constitution Check" section verifying principle alignment
- Quarterly review: check if any principles are routinely violated or need refinement

### Version Semantics

- **MAJOR**: Removal or redefinition of a principle (e.g., allowing globally-mutable state)
- **MINOR**: New principle or material guidance expansion (e.g., adding observability requirements)
- **PATCH**: Clarifications, typo fixes, wording improvements (no behavioral change)

---

**Version**: 1.0.0 | **Ratified**: 2025-10-29 | **Last Amended**: 2025-10-29
