# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. It uses Claude AI (via Anthropic API) to generate React components based on user descriptions, with a live preview system and code editor.

## Development Commands

```bash
# First-time setup (install dependencies + initialize database)
npm run setup

# Start development server with Turbopack
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run linter
npm run lint

# Run tests
npm test

# Reset database (destructive!)
npm run db:reset
```

## Testing

This project uses Vitest for testing. Test files are located in `__tests__` directories alongside components:
- Component tests: `src/components/**/__tests__/*.test.tsx`
- Library tests: `src/lib/**/__tests__/*.test.ts`

To run a single test file:
```bash
npm test -- path/to/test-file.test.tsx
```

## Architecture

### Virtual File System
The core architecture revolves around a **VirtualFileSystem** (`src/lib/file-system.ts`) that maintains React components entirely in memory—no files are written to disk during development. The file system:
- Uses a Map-based tree structure with FileNode objects
- Supports standard operations: create, read, update, delete, rename
- Serializes to JSON for database persistence
- Integrates with AI tools for component generation

### AI Integration Flow
1. **Chat API** (`src/app/api/chat/route.ts`) receives user messages and file system state
2. **Vercel AI SDK** streams responses from Claude with tool calling enabled
3. **AI Tools**:
   - `str_replace_editor` (`src/lib/tools/str-replace.ts`): Create, view, and edit files using string replacement
   - `file_manager` (`src/lib/tools/file-manager.ts`): Rename and delete files/directories
4. **System Prompt** (`src/lib/prompts/generation.tsx`): Instructs AI to create React components with Tailwind CSS, always starting with `/App.jsx` as entry point
5. **File System Updates**: Tool executions modify the VirtualFileSystem
6. **Auto-save**: On completion, authenticated users' projects are saved to database

### Preview System
The preview (`src/components/preview/PreviewFrame.tsx`) uses an innovative approach:
- **JSX Transformer** (`src/lib/transform/jsx-transformer.ts`): Converts JSX files to ES modules using Babel Standalone
- **Import Maps**: Creates browser import maps with blob URLs for each transformed module
- **Live Rendering**: Sandboxed iframe loads modules via import maps and renders React components
- Entry point auto-detection: Looks for `/App.jsx`, `/App.tsx`, `/index.jsx`, or `/index.tsx`

### Authentication & Persistence
- **JWT-based auth** (`src/lib/auth.ts`): Session management with jose library
- **Prisma with SQLite**: User accounts and project storage
- **Anonymous mode**: Users can use the app without signing up (no persistence)
- **Anonymous work tracking** (`src/lib/anon-work-tracker.ts`): Tracks unsaved work for anonymous users

### Mock Provider
When `ANTHROPIC_API_KEY` is not set, the app uses a **MockLanguageModel** (`src/lib/provider.ts`) that returns static component examples. This enables development and testing without API costs.

### Context Architecture
React Context providers manage global state:
- **FileSystemContext** (`src/lib/contexts/file-system-context.tsx`): Wraps VirtualFileSystem for React components
- **ChatContext** (`src/lib/contexts/chat-context.tsx`): Manages chat messages and AI streaming state

## Key Conventions

### Import Paths
All non-library imports use the `@/` alias pointing to the `src` directory:
```tsx
import { VirtualFileSystem } from '@/lib/file-system';
import { ChatInterface } from '@/components/chat/ChatInterface';
```

### Generated Components
AI-generated components should:
- Use `/App.jsx` as the main entry point (required)
- Import other components using `@/` alias
- Style with Tailwind CSS classes (no inline styles)
- Export default component from each file

### Database Migrations
When modifying the Prisma schema:
```bash
npx prisma migrate dev --name descriptive_migration_name
npx prisma generate
```

## Tech Stack

- **Framework**: Next.js 15 with App Router
- **React**: Version 19
- **AI**: Anthropic Claude (claude-sonnet-4-0) via Vercel AI SDK
- **Database**: Prisma with SQLite
- **Styling**: Tailwind CSS v4
- **Testing**: Vitest with Testing Library
- **UI Components**: Radix UI primitives (Dialog, Tabs, Popover, etc.)
- **Editor**: Monaco Editor (VS Code's editor)
- **Transformer**: Babel Standalone for JSX→ESM conversion

## Environment Variables

Optional `.env` file:
```
ANTHROPIC_API_KEY=your-api-key-here  # Optional; uses mock provider if absent
JWT_SECRET=your-jwt-secret           # Defaults to 'development-secret-key'
```
