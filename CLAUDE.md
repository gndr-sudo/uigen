# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. Users describe components in chat, Claude generates React/JSX code, and components render live in a sandboxed iframe.

## Commands

```bash
npm run dev          # Start dev server (http://localhost:3000)
npm run build        # Production build
npm run start        # Start production server
npm run lint         # ESLint
npm run test         # Run all tests with Vitest
npx vitest run src/lib/__tests__/file-system.test.ts  # Run single test file
npm run setup        # Install deps + Prisma generate + migrate
npm run db:reset     # Reset database
```

## Architecture

### Core Data Flow

```
User Chat Input → /api/chat → Claude AI (with tools) → Tool Calls → FileSystemContext → PreviewFrame (iframe)
```

1. **Chat Interface** sends messages to `/api/chat`
2. **Claude** receives file system state + messages, generates code via tool calls
3. **Tool calls** (`str_replace_editor`, `file_manager`) update the virtual file system
4. **PreviewFrame** transforms JSX with Babel and renders in sandboxed iframe

### Key Directories

- `src/app/` - Next.js App Router pages and API routes
- `src/components/chat/` - Chat UI (MessageList, MessageInput)
- `src/components/editor/` - Code editor (Monaco) and FileTree
- `src/components/preview/` - Live preview iframe with Babel transformation
- `src/lib/contexts/` - FileSystemContext and ChatContext providers
- `src/lib/tools/` - AI tool definitions for file operations
- `src/actions/` - Server actions for auth (sign-up, sign-in, sign-out)

### Virtual File System

`src/lib/file-system.ts` - In-memory file system class that:
- Stores files as `FileNode` tree structure
- Serializes to JSON for database persistence and API transmission
- Entry point is always `/App.jsx`

### Authentication

- JWT tokens in HttpOnly cookies (7-day expiration)
- `src/lib/auth.ts` - Token creation/verification
- `src/actions/index.ts` - Server actions for auth operations
- Anonymous users can create components; work persists to project on sign-in

### Database

SQLite with Prisma ORM. Schema defined in `prisma/schema.prisma` - reference it anytime you need to understand the structure of data stored in the database.

Key models:
- `User` - email, hashed password, projects
- `Project` - name, messages (JSON), data (JSON file system)

Messages and file system state stored as JSON strings.

### AI Integration

- Model: `claude-haiku-4-5` via `@ai-sdk/anthropic`
- Falls back to `MockLanguageModel` if no `ANTHROPIC_API_KEY`
- System prompt in `src/lib/prompts/generation.tsx`
- Two tools: `str_replace_editor` (file ops), `file_manager` (rename/delete)

### Preview Rendering

`src/components/preview/PreviewFrame.tsx`:
- Transforms JSX/TS to JS using Babel Standalone
- Creates import maps for module resolution
- Renders in sandboxed iframe with blob URLs

## Testing

Vitest + React Testing Library + JSDOM. Tests located in `__tests__/` subdirectories.

## Code Style

- Use comments sparingly. Only comment complex code.

## Environment Variables

- `ANTHROPIC_API_KEY` - Optional; uses mock provider if missing
- `JWT_SECRET` - For signing session tokens
- `DATABASE_URL` - SQLite path (default: prisma/dev.db)
