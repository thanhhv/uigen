# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. Users describe components in a chat interface, Claude generates the code, and a sandboxed iframe renders it in real-time using an in-memory virtual file system.

## Commands

```bash
npm run setup          # Install deps + generate Prisma client + run migrations
npm run dev            # Dev server with Turbopack (localhost:3000)
npm run build          # Production build
npm run lint           # ESLint
npm test               # Vitest (all tests)
npx vitest run src/lib/__tests__/file-system.test.ts  # Single test file
npm run db:reset       # Reset SQLite database
npx prisma studio      # Database GUI
```

## Architecture

**Next.js 15 App Router** with React 19, TypeScript, Tailwind CSS v4, Prisma (SQLite).

### Key Layers

- **`src/app/`** — App Router pages and API routes. Dynamic route `[projectId]` loads saved projects. `/api/chat/route.ts` handles AI streaming via Vercel AI SDK.
- **`src/actions/`** — Server actions ("use server") for auth (signUp/signIn/signOut) and project CRUD.
- **`src/components/`** — Feature-grouped: `auth/`, `chat/`, `editor/`, `preview/`, plus `ui/` (Shadcn/Radix primitives, new-york style).
- **`src/lib/contexts/`** — Two React contexts drive the app:
  - `FileSystemProvider` — wraps a `VirtualFileSystem` class (`lib/file-system.ts`) that stores all generated files in memory.
  - `ChatProvider` — wraps Vercel AI SDK's `useChat` hook for streaming AI responses.
- **`src/lib/tools/`** — AI tool definitions (`str_replace_editor`, `file_manager`) that operate on the virtual file system during generation.
- **`src/lib/transform/`** — Babel-based JSX/TSX transformer that converts code for in-browser execution via blob URLs and import maps (React loaded from esm.sh CDN).
- **`src/lib/prompts/generation.tsx`** — System prompt for AI generation.
- **`src/lib/provider.ts`** — LLM provider; uses Anthropic Claude Haiku 4.5 when `ANTHROPIC_API_KEY` is set, otherwise returns static mock responses.
- **`src/lib/auth.ts`** — JWT session management (jose library, `auth-token` cookie).

### Live Preview Pipeline

Files in virtual file system → Babel transforms JSX/TSX in browser → blob URLs with import maps → sandboxed iframe renders components with React from esm.sh CDN.

### Database

Prisma with SQLite (`prisma/dev.db`). Two models: `User` (email/password with bcrypt) and `Project` (serialized messages + file system data). Projects support anonymous ownership.

## Environment Variables

- `ANTHROPIC_API_KEY` — Optional. Without it, the app uses a mock provider with static responses.
- `JWT_SECRET` — Optional. Defaults to `"development-secret-key"`.

## Code Style

- Use comments sparingly. Only comment complex code.

## Path Alias

`@/*` maps to `./src/*` (configured in tsconfig.json).

## Testing

Vitest with jsdom environment and React Testing Library. Tests are co-located in `__tests__/` directories next to the code they test.
