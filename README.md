# Media Toolbox

> A complete solution for downloading, storing, and managing online videos locally

## 🎯 Overview

Media Toolbox is a monorepo workspace that simplifies the process of downloading and organizing online videos. It consists of three integrated components:

- **Backend Service** - Production-ready Go server for video downloads
- **Browser Extension** - Chrome extension for one-click video capture
- **PWA** (planned) - Progressive web app for viewing and managing your media library

Built with performance, reliability, and developer experience in mind.

## 🏗️ Architecture

```
media-toolbox/
├── videofetch/          # Go backend service (production-ready)
│   ├── cmd/             # Main application entry point
│   ├── internal/        # Core business logic
│   │   ├── config/      # Configuration management
│   │   ├── download/    # Download manager, workers, yt-dlp integration
│   │   ├── server/      # HTTP server, REST API handlers
│   │   ├── store/       # SQLite database layer
│   │   ├── ui/          # Templ templates + HTMX dashboard
│   │   └── logging/     # Structured JSON logging
│   └── static/          # CSS, assets
├── toolbox-ext/         # Chrome extension (WXT + SolidJS scaffold)
│   └── entrypoints/     # Background, content scripts, popup
└── web/                 # PWA (not yet implemented)
```

### Component Interaction

```
┌─────────────────┐      URLs       ┌──────────────────┐
│     Chrome      │ ──────────────> │    videofetch    │
│   Extension     │                 │   Go Backend     │
└─────────────────┘                 │                  │
                                    │  • REST API      │
┌─────────────────┐                 │  • yt-dlp        │
│   PWA (future)  │ <────────────── │  • SQLite DB     │
│  Media Viewer   │    metadata     │  • Dashboard     │
└─────────────────┘                 └──────────────────┘
                                            │
                                            v
                                    ┌──────────────────┐
                                    │  ~/Videos/       │
                                    │  videofetch/     │
                                    └──────────────────┘
```

## ⚡ Quick Start

### Prerequisites

- [mise](https://mise.jdx.dev/) - Development tool manager
- [yt-dlp](https://github.com/yt-dlp/yt-dlp) - Video downloader (must support `--progress-template`)

### Bootstrap

```bash
# Clone with submodules
git clone --recursive https://github.com/becksclair/media-toolbox.git
cd media-toolbox

# Install toolchains and dependencies
mise run bootstrap
```

### Running Components

```bash
# Backend API server
mise run dev:api

# Chrome extension (development mode)
mise run dev:ext:watch

# Load extension in Chrome
mise run chrome:dev
```

Visit `http://localhost:8080/dashboard` to access the web UI.

## 🧩 Components

### videofetch - Backend Service

**Status**: Production-ready

A high-performance Go web service that handles video downloads via `yt-dlp`.

**Features**:
- ✅ REST API for single and batch downloads
- ✅ Real-time progress tracking with WebSocket-like updates
- ✅ SQLite database for download history and metadata
- ✅ Web dashboard (Templ + HTMX, no build step)
- ✅ Worker pool with bounded queue and graceful shutdown
- ✅ Rate limiting (60 req/min per IP)
- ✅ Structured JSON logging
- ✅ Automatic metadata extraction (title, duration, thumbnails)

**API Endpoints**:
- `POST /api/download_single` - Enqueue single video
- `POST /api/download` - Batch enqueue
- `GET /api/status` - Real-time download status
- `GET /api/downloads` - Historical downloads with filters
- `GET /healthz` - Health check

[Full Documentation →](./videofetch/README.md)

### toolbox-ext - Chrome Extension

**Status**: Scaffold (WIP)

Browser extension for capturing video URLs and sending them to the backend.

**Tech Stack**: WXT, SolidJS, TypeScript

**Planned Features**:
- 🚧 Detect video pages automatically
- 🚧 One-click download button overlay
- 🚧 Context menu integration
- 🚧 Batch URL capture from tabs
- 🚧 Extension popup with download status

### web - PWA (Planned)

**Status**: Not yet implemented

A progressive web app for browsing and managing downloaded videos.

**Planned Features**:
- 📋 Grid/list view of media library
- 🔍 Search and filtering
- 🏷️ Tagging and collections
- 📊 Storage analytics
- ▶️ Built-in video player
- 📱 Mobile-responsive design

## 🛠️ Tech Stack

| Component      | Technologies                                      |
| -------------- | ------------------------------------------------- |
| **Backend**    | Go 1.23+, SQLite, yt-dlp, Templ, HTMX, Tailwind  |
| **Extension**  | WXT, SolidJS, TypeScript, Bun                     |
| **PWA**        | TBD (likely Vite + SolidJS)                       |
| **Tooling**    | mise, oxlint, golangci-lint                       |
| **Dev Tools**  | VS Code multi-root workspace                      |

## 📦 Development

### Workspace Structure

This is a VS Code multi-root workspace with three folders:
- `media-toolbox` (root)
- `videofetch` (submodule)
- `toolbox-ext` (submodule)

Open `media-toolbox.code-workspace` in VS Code for the best development experience.

### Available Commands

```bash
# Backend
mise run dev:api           # Run videofetch server
mise run test:api          # Run Go tests
mise run lint:api          # Lint Go code

# Extension
mise run dev:ext:watch     # Build extension in watch mode
mise run lint:ext          # Lint TypeScript code
mise run fmt:ext           # Format code with oxfmt

# Chrome
mise run chrome:dev        # Launch Chrome with extension loaded
```

### Git Submodules

Both `videofetch` and `toolbox-ext` are Git submodules with their own repositories:
- [videofetch](https://github.com/becksclair/videofetch)
- [toolbox-ext](https://github.com/becksclair/toolbox-ext)

**Working with submodules**:

```bash
# Update submodules to latest
git submodule update --remote

# Make changes in a submodule
cd videofetch
git checkout -b feature/my-feature
# ... make changes ...
git commit -m "Add feature"
git push origin feature/my-feature

# Update parent repo to point to new submodule commit
cd ..
git add videofetch
git commit -m "Update videofetch submodule"
```

## 🚀 Roadmap

### Current State
- ✅ Backend service fully functional
- ✅ API, database, and dashboard working
- 🚧 Extension is scaffold only

### Next Steps
1. **Extension MVP** (Phase 1)
   - Implement URL detection and capture
   - Add one-click download button
   - Create functional popup UI
   - Connect to backend API

2. **PWA Foundation** (Phase 2)
   - Set up Vite + SolidJS project
   - Design media grid layout
   - Implement search and filtering
   - Connect to backend API

3. **Enhancement** (Phase 3)
   - Advanced metadata editing
   - Playlist support
   - Video transcoding options
   - Cloud sync (optional)

## 🧪 Testing

```bash
# Backend unit tests
cd videofetch
go test ./... -race

# Backend integration tests (requires network)
go test -tags=integration ./internal/integration -v

# Extension (when implemented)
cd toolbox-ext
bun test
```

## 🤝 Contributing

1. **Choose a component** - Backend, extension, or PWA
2. **Check the submodule** - Each component has its own repo
3. **Create a branch** - Work in the submodule's repository
4. **Test thoroughly** - Run tests before committing
5. **Update parent** - Point parent repo to new submodule commit

See individual component READMEs for specific contribution guidelines.

## 📄 License

[License information to be added]

## 🙏 Acknowledgments

- [yt-dlp](https://github.com/yt-dlp/yt-dlp) - The amazing video downloader
- [WXT](https://wxt.dev/) - Next-gen web extension framework
- [Templ](https://templ.guide/) - Type-safe Go templates
- [HTMX](https://htmx.org/) - High-power tools for HTML

---

Built with ❤️ for local media archiving
