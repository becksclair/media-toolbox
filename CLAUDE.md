# Media Toolbox - AI Agent Instructions

## Project Overview

Media Toolbox is a monorepo workspace for downloading, storing, and managing online videos locally. It combines three components:
1. **videofetch** - Production-ready Go backend service
2. **toolbox-ext** - Chrome extension (WXT + SolidJS, currently scaffold)
3. **web** - PWA for media management (planned, not yet implemented)

Both videofetch and toolbox-ext are Git submodules with independent repositories.

## Architecture

### Monorepo Structure

```
/home/bex/projects/media-toolbox/
├── videofetch/              # Git submodule
│   ├── cmd/videofetch/      # Main entry point
│   ├── internal/
│   │   ├── config/          # CLI flags, configuration
│   │   ├── download/        # Core: manager, workers, yt-dlp integration
│   │   ├── server/          # HTTP server, API handlers, middleware
│   │   ├── store/           # SQLite database operations
│   │   ├── ui/              # Templ templates (.templ files)
│   │   └── logging/         # Structured JSON logger
│   ├── static/              # CSS (Tailwind v4), assets
│   ├── go.mod
│   └── Makefile
├── toolbox-ext/             # Git submodule
│   ├── entrypoints/
│   │   ├── background.ts    # Extension background service worker
│   │   ├── content.ts       # Content script (currently google.com only)
│   │   └── popup/           # Popup UI (SolidJS components)
│   ├── wxt.config.ts
│   ├── package.json
│   └── tsconfig.json
├── .mise.toml               # Tool versions + tasks
├── media-toolbox.code-workspace
└── README.md
```

### Component Interaction

1. **Chrome Extension** → Captures video URLs from web pages
2. **Backend API** → Receives URLs, queues downloads, stores metadata
3. **yt-dlp** → Downloads videos to `~/Videos/videofetch`
4. **SQLite DB** → Persists download history and metadata
5. **Dashboard** → Web UI for monitoring and managing downloads
6. **PWA (future)** → Browse and play downloaded videos

## Technology Stack

### videofetch (Go Backend)

| Technology         | Purpose                                      |
| ------------------ | -------------------------------------------- |
| **Go 1.23+**       | Main language                                |
| **modernc.org/sqlite** | Pure-Go SQLite driver                     |
| **Templ**          | Type-safe HTML templates (github.com/a-h/templ) |
| **HTMX**           | Dynamic UI updates without client JS        |
| **Tailwind v4**    | CSS framework (built via Bun)               |
| **yt-dlp**         | External video downloader (system dependency) |

### toolbox-ext (Chrome Extension)

| Technology      | Purpose                                      |
| --------------- | -------------------------------------------- |
| **WXT**         | Web extension framework                      |
| **React**       | UI library (popup UI only)                   |
| **TypeScript**  | Type-safe JavaScript                         |
| **Bun**         | Fast JS runtime and package manager          |

**Note**: README.md mentions SolidJS, but the extension currently uses React. Keep React for now.

### Development Tooling

| Tool             | Purpose                                      |
| ---------------- | -------------------------------------------- |
| **mise**         | Tool version manager (replaces asdf)         |
| **oxlint**       | Fast TypeScript linter                       |
| **golangci-lint**| Go linter aggregator                         |
| **VS Code**      | Multi-root workspace setup                   |

## Quick Command Reference

For most common tasks:

```bash
# Full setup (first time)
mise run bootstrap              # Init submodules + install tools + deps

# Typical dev loop
mise run dev:api                # Terminal 1: Backend on :8080
mise run dev:ext:watch          # Terminal 2: Extension with hot reload
mise run chrome:dev             # Terminal 3: Chrome with extension

# Verify before commit
mise run test:api               # Go tests with race detector
mise run lint:api               # Go linting
mise run lint:ext               # TypeScript linting
mise run fmt:ext                # Auto-format TypeScript
```

## Full Build/Test/Run Commands

### Workspace-level Commands (via mise)

```bash
# Bootstrap entire workspace
mise run bootstrap              # Init submodules + install tools + deps

# Backend
mise run dev:api                # Run videofetch on :8080
mise run test:api               # go test ./... -race
mise run lint:api               # golangci-lint run

# Extension
mise run dev:ext:watch          # WXT dev server with hot reload
mise run lint:ext               # oxlint with type checking
mise run fmt:ext                # oxfmt formatter

# Chrome debugging
mise run chrome:dev             # Launch Chrome with extension loaded (remote port 9222)
```

### videofetch-specific Commands

```bash
cd videofetch

# Build
go build -o videofetch ./cmd/videofetch
make build                      # Same as above

# Run
./videofetch --port 8080 --host 0.0.0.0 --output-dir ~/Videos/videofetch
make run                        # Default flags

# Test
go test ./... -race             # All unit tests
go test -tags=integration ./internal/integration -v  # Integration tests
go test -run TestName ./internal/package             # Single test
go test -coverprofile=coverage.out ./...             # Coverage

# Templates
make generate                   # Regenerate Templ templates (after editing .templ)
make tools                      # Install Templ CLI

# CSS
bun run build-css               # Rebuild Tailwind to ./static/style.css

# Lint
golangci-lint run ./...
```

### toolbox-ext-specific Commands

```bash
cd toolbox-ext

# Development
bun dev                         # WXT dev mode (Chrome by default)
bun dev:firefox                 # WXT dev mode for Firefox

# Build
bun build                       # Production build to ./.wxt/dist
bun zip                         # Create distributable zip

# Lint/Format
bun lint                        # oxlint with all plugins
bun fmt                         # oxfmt formatter
bun compile                     # TypeScript type check only
```

**Note**: `.wxt/` directory is generated by WXT build process. Don't edit files here directly; edit source files in `entrypoints/` and `public/` instead.

## Code Style and Conventions

### Go (videofetch)

- **Error Handling**: Use stable error message strings like `invalid_request`, `queue_full`, `yt_dlp_not_found`
- **Dependency Injection**: Pass dependencies explicitly (no globals). Use interfaces for abstraction.
- **Concurrency**: Use bounded channels/queues. Worker pools with configurable concurrency.
- **Logging**: Structured JSON logs with event types (`download_start`, `http_request`, etc.)
- **Configuration**: Flag-driven features. Use `internal/config` package for CLI flags.
- **Progress**: Monotonically increasing (0-100%), never decrease
- **Database**: All structs use JSON tags for API responses. Use prepared statements.
- **Testing**: Table-driven tests. Integration tests use build tag `//go:build integration`

**Import Groups** (in order):
1. Standard library
2. External packages
3. Internal packages

**Naming**:
- Interfaces: Descriptive names (e.g., `Downloader`, `Store`, `Logger`)
- Errors: Lowercase with underscores (e.g., `queue_full`)
- HTTP handlers: `Handle*` prefix (e.g., `HandleDownload`, `HandleStatus`)

### TypeScript (toolbox-ext)

- **Framework**: Follow WXT conventions for entrypoints
- **Components**: SolidJS with TypeScript
- **Linting**: Use oxlint (stricter than ESLint)
- **Formatting**: oxfmt (Rust-based, fast)
- **File Organization**:
  - `entrypoints/` - Background, content scripts, popup
  - `assets/` - Images, icons
  - `public/` - Static files copied to output

## Development Workflows

### Starting Fresh

```bash
git clone --recursive git@github.com:becksclair/media-toolbox.git
cd media-toolbox
mise run bootstrap
```

### Daily Development

```bash
# Terminal 1: Backend
mise run dev:api

# Terminal 2: Extension
mise run dev:ext:watch

# Terminal 3: Chrome with extension
mise run chrome:dev
```

Visit `http://localhost:8080/dashboard` for backend UI.

### Adding Features to videofetch

1. **Locate code**: Use `internal/` package structure
   - API handlers → `internal/server/`
   - Download logic → `internal/download/`
   - Database → `internal/store/`
   - Templates → `internal/ui/`

2. **Make changes**: Follow Go conventions

3. **Update templates** (if needed):
   ```bash
   # Edit .templ files
   make generate
   ```

4. **Test**:
   ```bash
   go test ./... -race
   go test -tags=integration ./internal/integration -v
   ```

5. **Build**:
   ```bash
   make build
   ./videofetch --port 8080
   ```

### Adding Features to toolbox-ext

1. **Identify entrypoint**:
   - Background logic → `entrypoints/background.ts`
   - Content script (page interaction) → `entrypoints/content.ts`
   - Popup UI → `entrypoints/popup/App.tsx`

2. **Make changes**: Use SolidJS patterns

3. **Test**: Hot reload is automatic in dev mode

4. **Build**:
   ```bash
   bun build
   ```

### Working with Git Submodules

Both `videofetch` and `toolbox-ext` are submodules:

```bash
# Update submodules to latest upstream
git submodule update --remote

# Make changes in submodule
cd videofetch
git checkout -b feature/my-feature
# ... edit files ...
git add .
git commit -m "Add my feature"
git push origin feature/my-feature

# Update parent to reference new commit
cd ..
git add videofetch
git commit -m "Update videofetch submodule to include my-feature"
git push
```

**Important**: Always commit in the submodule first, then update the parent reference.

## Testing Strategy

### videofetch

| Type               | Command                                           | Coverage                          |
| ------------------ | ------------------------------------------------- | --------------------------------- |
| **Unit**           | `go test ./... -race`                             | Handlers, manager, store, config  |
| **Integration**    | `go test -tags=integration ./internal/integration -v` | Real yt-dlp, network, DB         |
| **Coverage**       | `go test -coverprofile=coverage.out ./...`        | Generate coverage report          |

**Integration Tests**:
- Require network and yt-dlp installed
- Use real URLs (configurable via env vars)
- Test metadata extraction, DB persistence, download workflows

**Environment Overrides**:
- `INTEGRATION_URL=https://...` - Single test URL
- `INTEGRATION_URLS="https://u1, https://u2"` - Multiple URLs

### toolbox-ext

Currently no tests (scaffold phase). Future tests will use:
- Unit: Vitest
- E2E: Playwright + WXT testing utils

## Key Patterns and Abstractions

### videofetch Architecture Patterns

1. **Download Manager** (`internal/download/manager.go`)
   - Worker pool with configurable concurrency (`--workers` flag)
   - Bounded queue with backpressure (`--queue` flag)
   - Graceful shutdown: drain queue, cancel in-flight downloads
   - Registry for tracking active downloads

2. **Progress Tracking**
   - Parse yt-dlp output using custom `--progress-template`
   - Update both in-memory state and database
   - Real-time streaming via HTMX polling

3. **Database Worker** (`internal/download/dbworker.go`)
   - Background goroutine for async DB updates
   - Buffered channel for update queue
   - Prevents blocking download workers

4. **Structured Logging**
   - JSON lines to stdout
   - Event types for filtering (`http_request`, `download_complete`, etc.)
   - Context-aware (request IDs, download IDs)

5. **Rate Limiting**
   - Per-IP rate limiter (60 req/min)
   - Applied at middleware layer

6. **Graceful Shutdown**
   - HTTP server stops accepting requests
   - Drain existing connections
   - Cancel context for all workers
   - Close database cleanly

### Extension Patterns (Future)

1. **Message Passing**: Background ↔ Content Script ↔ Popup
2. **Storage**: `browser.storage.local` for settings/state
3. **Permissions**: Minimal scope (activeTab, storage)

## Common Tasks

### Debugging

**Backend**:
```bash
# Enable debug logging
./videofetch --log-level debug

# Use VS Code debugger
# See .vscode/launch.json
```

**Extension**:
```bash
# Open Chrome DevTools
# Extension popup: right-click → Inspect
# Background: chrome://extensions → Details → Inspect views: background page
# Content script: Regular page DevTools → Console shows content script logs
```

### Database Operations

**Location**:
- Default: `~/.cache/videofetch/videofetch.db` (Linux/macOS)
- Custom: Use `--db` flag

**Schema**: See `videofetch/internal/store/store.go` for table definitions. Main tables:
- `downloads` - Download records with URL, title, status, progress
- Other supporting tables for metadata and tracking

**Query manually**:
```bash
sqlite3 ~/.cache/videofetch/videofetch.db "SELECT id, url, status FROM downloads;"

# Or reset the database
rm ~/.cache/videofetch/videofetch.db  # Will be recreated on next run
```

### Adding API Endpoints

1. Define handler in `internal/server/handlers.go`
2. Register route in `internal/server/server.go` `setupRoutes()`
3. Add tests in `internal/server/handlers_test.go`
4. Document in `videofetch/README.md`

### Adding Dashboard UI

1. Create `.templ` file in `internal/ui/`
2. Run `make generate` to compile to Go
3. Wire up route in `internal/server/server.go`
4. Update CSS if needed: `bun run build-css`

## Important Constraints

### videofetch

- **yt-dlp dependency**: Must be installed and support `--progress-template`. Checked at startup.
- **SQLite**: Pure-Go driver, no CGO required
- **No client-side JS**: Dashboard uses HTMX for interactivity
- **Bounded resources**: Queue has max capacity, configurable via `--queue`
- **Idempotent operations**: Same URL can be re-downloaded (new DB entry created)

### toolbox-ext

- **Manifest V3**: Use service workers, not persistent background pages
- **WXT conventions**: Entrypoints auto-discovered from `entrypoints/`
- **Browser compatibility**: Chrome primary, Firefox secondary

## Environment Variables

### videofetch

None by default. Configuration via CLI flags only.

**Integration Tests**:
- `INTEGRATION_URL` - Override test URL
- `INTEGRATION_URLS` - Override test URLs (comma-separated)

### toolbox-ext

None currently. Future: backend URL configuration.

## Project-Specific Context

### Why Git Submodules?

- **Independent versioning**: Each component has its own release cycle
- **Separate repositories**: videofetch and toolbox-ext can be used standalone
- **Modular development**: Teams can work on components independently

### Why Templ + HTMX?

- **No build step**: Server-rendered HTML, no webpack/vite for dashboard
- **Simplicity**: Minimal JavaScript, fast page loads
- **Type safety**: Templ provides compile-time template checking

### Why WXT for Extension?

- **Modern DX**: Hot module reload, TypeScript, framework support
- **Multi-browser**: Single codebase for Chrome, Firefox, Safari
- **Manifest V3 ready**: Built for latest extension standards

### Why SQLite?

- **Embedded**: No separate database server
- **Sufficient scale**: Handles thousands of downloads easily
- **Portability**: Single file, easy backups
- **Pure Go**: No CGO, cross-platform compilation

## Future Considerations

### PWA Implementation

When implementing the PWA (`web/`):
1. Use Vite + SolidJS for consistency with extension
2. Connect to videofetch REST API
3. Consider local-first architecture (cache downloads list)
4. Implement video player with HLS support
5. Add offline capabilities (service worker)

### Extension MVP

Priority features:
1. Detect video pages (YouTube, Vimeo, etc.)
2. Context menu "Download with VideoFetch"
3. Popup UI with download status
4. Settings page for backend URL configuration
5. Batch download from open tabs

### Scaling Considerations

If videofetch needs to scale:
- Replace SQLite with PostgreSQL
- Add Redis for queue/cache
- Horizontal scaling behind load balancer
- Object storage (S3) for video files

## Quick Reference

### File Locations

| What                  | Path                                          |
| --------------------- | --------------------------------------------- |
| Backend entry point   | `videofetch/cmd/videofetch/main.go`           |
| API handlers          | `videofetch/internal/server/handlers.go`      |
| Download manager      | `videofetch/internal/download/manager.go`     |
| Database layer        | `videofetch/internal/store/store.go`          |
| Templates             | `videofetch/internal/ui/*.templ`              |
| Extension popup       | `toolbox-ext/entrypoints/popup/App.tsx`       |
| Extension background  | `toolbox-ext/entrypoints/background.ts`       |
| Extension content     | `toolbox-ext/entrypoints/content.ts`          |
| Workspace config      | `media-toolbox.code-workspace`                |
| Tool versions         | `.mise.toml`                                  |

### URLs

| Service           | URL                                     |
| ----------------- | --------------------------------------- |
| Backend dashboard | `http://localhost:8080/dashboard`       |
| API base          | `http://localhost:8080/api`             |
| Health check      | `http://localhost:8080/healthz`         |
| Chrome debugger   | `chrome://extensions` (Developer mode)  |

### Default Paths

| Resource           | Default Path                              |
| ------------------ | ----------------------------------------- |
| Downloads          | `~/Videos/videofetch/`                    |
| Database (Linux)   | `~/.cache/videofetch/videofetch.db`       |
| Database (macOS)   | `~/.cache/videofetch/videofetch.db`       |
| Database (Windows) | `%APPDATA%/videofetch/videofetch.db`      |

## Troubleshooting

### yt-dlp not found

```bash
# Install yt-dlp
pip install yt-dlp
# or
brew install yt-dlp

# Verify
yt-dlp --version
yt-dlp --progress-template "test"
```

### Submodule issues

```bash
# Submodules not initialized
git submodule update --init --recursive

# Submodule detached HEAD
cd videofetch
git checkout main
git pull
```

### Extension not loading

```bash
# Rebuild extension
cd toolbox-ext
bun build

# Check manifest
cat .wxt/dist/manifest.json
```

### Database locked

```bash
# Close all videofetch instances
pkill videofetch

# Verify no locks
lsof ~/.cache/videofetch/videofetch.db
```

## Additional Resources

- videofetch README: `./videofetch/README.md`
- videofetch Agent Instructions: `./videofetch/AGENTS.md`
- WXT Documentation: https://wxt.dev/
- Templ Documentation: https://templ.guide/
- yt-dlp Documentation: https://github.com/yt-dlp/yt-dlp#readme
