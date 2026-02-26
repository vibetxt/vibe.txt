# .vibetxt - Minimalist Media Downloader

## Overview

.vibetxt is a minimalist, privacy-focused media downloader web application. Users paste a URL from supported platforms (YouTube, Twitter/X, Instagram, TikTok, Reddit, SoundCloud, etc.) and the app processes the download through community Cobalt instances. The app acts as a frontend that discovers and routes requests to available Cobalt API instances, providing configurable download settings for video quality, audio format, codec preferences, and platform-specific options.

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend (React + Vite)
- **Framework**: React 18 with TypeScript, bundled by Vite
- **Routing**: Wouter (lightweight client-side router) with three pages: Home (`/`), About (`/about`), Instances (`/instances`)
- **Styling**: Tailwind CSS v4 with CSS variables for theming (dark mode default), shadcn/ui component library (New York style)
- **State Management**: TanStack React Query for server state; local React state for form/settings
- **Animations**: Framer Motion for page transitions and UI animations
- **Fonts**: Inter (sans) and JetBrains Mono (mono) from Google Fonts
- **Location**: All frontend code lives in `client/src/`

### Backend (Express 5 + Node.js)
- **Framework**: Express 5 running on Node.js with TypeScript (compiled via tsx in dev, esbuild for production)
- **API Pattern**: REST API under `/api/` prefix
- **Key Endpoints**:
  - `POST /api/process` - Accepts a download request with URL and settings, proxies to a Cobalt instance
  - `GET /api/instances` - Returns list of available Cobalt processing servers
- **Cobalt Instance Discovery**: Fetches available instances from `https://instances.cobalt.best/api/instances.json`, caches for 5 minutes, filters for online HTTPS instances, prefers non-auth instances sorted by score
- **Storage**: Currently uses an empty in-memory storage class (`MemStorage`). No database tables are actively used despite Drizzle being configured.

### Shared Code
- **Location**: `shared/schema.ts`
- **Purpose**: Zod schemas for download request/response validation shared between client and server
- **Key Schemas**: `downloadRequestSchema` (URL, video quality, audio format, codec, platform-specific flags), `downloadResponseSchema` (status types: stream, redirect, picker, error)

### Database
- **ORM**: Drizzle ORM configured for PostgreSQL via `DATABASE_URL` environment variable
- **Current State**: Database is configured but no tables are defined in the schema. The schema file only contains Zod validation schemas, not Drizzle table definitions. The `MemStorage` class is empty.
- **Migration**: Use `npm run db:push` to push schema changes

### Build System
- **Development**: `npm run dev` starts the Express server with Vite middleware for HMR
- **Production Build**: `npm run build` runs a custom build script that uses Vite for the client and esbuild for the server, outputting to `dist/`
- **Production Start**: `npm start` serves the built files from `dist/`

### Dev vs Production Serving
- In development, Vite middleware handles the client with HMR (`server/vite.ts`)
- In production, Express serves static files from `dist/public` with SPA fallback (`server/static.ts`)

## External Dependencies

### Third-Party APIs
- **Cobalt Instances API**: `https://instances.cobalt.best/api/instances.json` - Community-maintained list of Cobalt media processing servers. The app discovers available instances and routes download requests to them.

### Database
- **PostgreSQL**: Required via `DATABASE_URL` environment variable. Used with Drizzle ORM, though no tables are currently defined. The `connect-pg-simple` package is included for potential session storage.

### Key NPM Packages
- **UI**: shadcn/ui components (Radix UI primitives), Tailwind CSS, Framer Motion, Lucide icons
- **Data**: TanStack React Query, Zod for validation, Drizzle ORM + drizzle-zod
- **Server**: Express 5, connect-pg-simple for PostgreSQL sessions
- **Build**: Vite, esbuild, tsx (TypeScript execution)

### Replit-Specific
- `@replit/vite-plugin-runtime-error-modal` - Runtime error overlay
- `@replit/vite-plugin-cartographer` and `@replit/vite-plugin-dev-banner` - Dev-only Replit integrations
- Custom `vite-plugin-meta-images` for OpenGraph image URL resolution on Replit deployments