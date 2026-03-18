# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. Users describe components in a chat interface, and the AI generates React code that is rendered in real-time.

## Tech Stack

- Next.js 15 with App Router and Turbopack
- React 19
- TypeScript 5
- Tailwind CSS v4
- Prisma with SQLite
- Anthropic Claude AI via Vercel AI SDK
- Monaco Editor for code editing
- Babel standalone for JSX transformation in preview

## Common Commands

```bash
# Development (requires Node 18+)
npm run dev                    # Start dev server on localhost:3000
npm run dev:daemon            # Start dev server in background with logs

# Building
npm run build                  # Build for production
npm run start                  # Start production server

# Testing
npm run test                   # Run all Vitest tests
npm run test -- <pattern>      # Run specific test file

# Linting
npm run lint                   # Run ESLint

# Database
npm run setup                  # Install deps, generate Prisma client, run migrations
npm run db:reset               # Reset database (destructive)
npx prisma migrate dev         # Create and apply new migration
npx prisma generate            # Regenerate Prisma client after schema changes
```

## Architecture

### Virtual File System

The core abstraction is `VirtualFileSystem` in `src/lib/file-system.ts`. Files are stored in memory only and never written to disk.

- Uses a Map-based structure with `FileNode` objects
- Supports CRUD operations, directory creation, file renaming
- Serialization/deserialization for database persistence
- React context at `src/lib/contexts/file-system-context.tsx` provides access throughout the component tree

### AI Integration

- API route at `src/app/api/chat/route.ts` handles streaming responses
- Uses Vercel AI SDK with tool calling for file operations
- Two tools available: `str_replace_editor` (create/edit files) and `file_manager` (rename/delete)
- If `ANTHROPIC_API_KEY` is not set, falls back to `MockLanguageModel` in `src/lib/provider.ts`
- System prompt defined in `src/lib/prompts/generation.tsx`

### Live Preview System

- `PreviewFrame` component renders generated code in an iframe
- `src/lib/transform/jsx-transformer.ts` creates import maps and HTML for the iframe
- Uses ES modules with import maps to resolve `@/` aliases
- Babel standalone transforms JSX in the browser

### Authentication

- Session-based auth using `jose` for JWT signing
- bcrypt for password hashing
- Session stored in HTTP-only cookie
- Auth actions in `src/actions/index.ts` (signUp, signIn, signOut, getUser)
- Middleware at `src/middleware.ts` protects `/api/projects` and `/api/filesystem` routes

### Data Model

Prisma schema (`prisma/schema.prisma`):
- User: id, email, password, projects[]
- Project: id, name, userId (optional), messages (JSON string), data (JSON string), timestamps
- Projects can be anonymous (userId is null) but only authenticated users can save/load them

### Node.js Compatibility

`node-compat.cjs` fixes Web Storage API issues with Node 25+ SSR by deleting `localStorage`/`sessionStorage` globals on the server.

## Key File Locations

| Purpose | Path |
|---------|------|
| File System | `src/lib/file-system.ts` |
| File System Context | `src/lib/contexts/file-system-context.tsx` |
| Chat Context | `src/lib/contexts/chat-context.tsx` |
| AI Provider | `src/lib/provider.ts` |
| Chat API | `src/app/api/chat/route.ts` |
| Auth Actions | `src/actions/index.ts` |
| Project Actions | `src/actions/create-project.ts`, `src/actions/get-project.ts`, `src/actions/get-projects.ts` |
| Preview | `src/components/preview/PreviewFrame.tsx` |
| JSX Transformer | `src/lib/transform/jsx-transformer.ts` |
| Str Replace Tool | `src/lib/tools/str-replace.ts` |
| File Manager Tool | `src/lib/tools/file-manager.ts` |
| Database Client | `src/lib/prisma.ts` |
| Auth Utils | `src/lib/auth.ts` |

## Environment Variables

```bash
ANTHROPIC_API_KEY=your-api-key-here    # Optional - falls back to mock provider if not set
```

## Code Style Guidelines

- Use comments only for complex code that isn't self-evident from reading
- Avoid comments for simple/straightforward logic

## Testing

Tests use Vitest with jsdom environment and React Testing Library.
- Test files: `**/__tests__/*.test.ts(x)`
- Config: `vitest.config.mts`
