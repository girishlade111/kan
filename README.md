<div align="center">
  <h1 align="center">Kan</h1>
  <p>The open-source project management alternative to Trello.</p>
</div>

<p align="center">
  <a href="https://kan.bn">Website</a> ·
  <a href="https://docs.kan.bn">Docs</a> ·
  <a href="https://kan.bn/kan/roadmap">Roadmap</a> ·
  <a href="https://discord.gg/e6ejRb6CmT">Discord</a>
</p>

<p align="center">
  <a href="LICENSE"><img alt="License" src="https://img.shields.io/badge/license-AGPLv3-purple"></a>
  <a href="https://nextjs.org"><img alt="Next.js" src="https://img.shields.io/badge/Next.js-black?logo=next.js"></a>
  <a href="https://www.typescriptlang.org"><img alt="TypeScript" src="https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white"></a>
  <a href="https://pnpm.io"><img alt="pnpm" src="https://img.shields.io/badge/pnpm-F69220?logo=pnpm&logoColor=white"></a>
  <a href="https://turbo.build"><img alt="Turborepo" src="https://img.shields.io/badge/Turborepo-EF4444?logo=turborepo&logoColor=white"></a>
</p>

---

## Table of Contents

- [Overview](#overview-)
- [Features](#features-)
- [Tech Stack](#tech-stack-️)
- [Architecture](#architecture-)
- [Project Structure](#project-structure-)
- [Quick Start — Local Development](#quick-start--local-development-️)
- [Available Scripts](#available-scripts-)
- [Environment Variables](#environment-variables-)
- [Self Hosting](#self-hosting-)
- [Testing](#testing-)
- [MCP Server (AI Control)](#mcp-server-ai-control-)
- [Contributing](#contributing-)
- [License](#license-)

## Overview 🚀

Kan is a full-featured, open-source project management tool and a modern alternative to Trello. It provides kanban boards, workspaces, team collaboration, activity tracking, templates, imports, and more — all self-hostable and built on a type-safe TypeScript monorepo.

This repository contains the complete source code for the Kan application: the Next.js web app, the tRPC API, the database layer, authentication, email templates, the MCP server for AI control, and end-to-end tests.

## Features 💫

- 👁️ **Board Visibility** — Control who can view and edit your boards (public / private)
- 🤝 **Workspace Members** — Invite members and collaborate with your team at different permission levels
- 🚀 **Trello Imports** — Easily import your existing Trello boards
- 🔍 **Labels & Filters** — Organise and find cards quickly
- 💬 **Comments** — Discuss and collaborate with your team directly on cards
- 📝 **Activity Log** — Track all card changes with a detailed, per-card activity history
- 🎨 **Templates** — Save time with reusable custom board templates
- 📎 **Attachments & Due Dates** — File uploads (S3-compatible storage), checklists, members, and due dates on cards
- 🔐 **Authentication** — Email/password plus Google, GitHub, Discord, and generic OIDC social login
- ⚡️ **Integrations (coming soon)** — Connect your favourite tools
- 🤖 **MCP Server** — Control your boards from any AI client via the Model Context Protocol

See the [roadmap](https://kan.bn/kan/roadmap) for upcoming features.

## Screenshot 👁️

<img width="1507" alt="hero-dark" src="https://github.com/user-attachments/assets/8490104a-cd5d-49de-afc2-152fd8a93119" />

## Tech Stack 🛠️

| Layer | Technology |
| --- | --- |
| Frontend | [Next.js](https://nextjs.org), [React](https://react.dev), [TypeScript](https://www.typescriptlang.org), [Tailwind CSS](https://tailwindcss.com) |
| API | [tRPC](https://trpc.io) (end-to-end type-safe procedures) |
| Database | [PostgreSQL](https://www.postgresql.org) with [Drizzle ORM](https://orm.drizzle.team) |
| Auth | [Better Auth](https://better-auth.com) (credentials + OAuth/OIDC) |
| Email | [React Email](https://react.email) + SMTP (Resend, etc.) |
| Payments | [Stripe](https://stripe.com) |
| Styling | Tailwind CSS + shared design tooling |
| Monorepo | [pnpm workspaces](https://pnpm.io) + [Turborepo](https://turbo.build) |
| Validation | [Zod](https://zod.dev) |
| AI | [Model Context Protocol](https://modelcontextprotocol.io) server (`@kan/mcp`) |
| Testing | Unit tests per workspace + [Playwright](https://playwright.dev) E2E |

## Architecture 🏗️

Kan is a **pnpm + Turborepo monorepo**. Every workspace is type-safe and shares TypeScript configs, ESLint configs, Prettier configs, and Tailwind presets from `tooling/`.

```
                        ┌──────────────────────┐
                        │      apps/web        │  Next.js UI (App Router)
                        └──────────┬───────────┘
                                   │ tRPC (HTTP)
                        ┌──────────▼───────────┐
                        │     packages/api     │  tRPC routers + Zod validation
                        └──────┬───────┬───────┘
               ┌───────────────┘       └───────────────┐
      ┌────────▼────────┐                    ┌─────────▼────────┐
      │  packages/db    │                    │  packages/auth   │
      │ Drizzle + repos │                    │    Better Auth   │
      └────────┬────────┘                    └──────────────────┘
               │
      ┌────────▼────────┐   ┌──────────────┐   ┌───────────────┐
      │   PostgreSQL    │   │ packages/    │   │ packages/     │
      └─────────────────┘   │ email        │   │ stripe / mcp  │
                            └──────────────┘   └───────────────┘
```

Key conventions:

- **Soft deletes** — entities use a `deletedAt` timestamp instead of hard deletes
- **Public IDs** — every user-facing entity exposes a 12-character `publicId`; internal DB IDs never leave the API
- **Index management** — cards carry sequential `index` fields maintained transactionally on create/move/delete
- **Activity logging** — every significant card change writes a row to `card_activity`
- **Authorization** — every protected procedure checks authentication and workspace membership (`assertUserInWorkspace`)

## Project Structure 📂

```
kan/
├── apps/
│   ├── web/                  # Next.js web application (UI, pages, components)
│   └── docs/                 # Documentation site
├── packages/
│   ├── api/                  # tRPC routers, procedures, Zod input schemas
│   ├── auth/                 # Better Auth configuration (credentials + OAuth/OIDC)
│   ├── db/                   # Drizzle schema, migrations, repositories
│   ├── email/                # React Email templates + SMTP sending
│   ├── e2e/                  # Playwright end-to-end tests
│   ├── logger/               # Shared @kan/logger (LOG_LEVEL-controlled)
│   ├── mcp/                  # Model Context Protocol server (@kan/mcp)
│   ├── shared/               # Shared utilities and types
│   └── stripe/               # Stripe billing integration
├── tooling/
│   ├── eslint/               # Shared ESLint config
│   ├── prettier/             # Shared Prettier config
│   ├── tailwind/             # Shared Tailwind preset
│   └── typescript/           # Shared tsconfig bases
├── cloud/                    # Cloud deployment compose files
├── docker-compose.yml        # Full self-host stack (web + migrate + postgres)
├── turbo.json                # Turborepo pipeline definitions
├── pnpm-workspace.yaml       # Workspace configuration
└── package.json              # Root scripts and shared devDependencies
```

## Quick Start — Local Development 🧑‍💻

### Prerequisites

| Tool | Version |
| --- | --- |
| Node.js | >= 20.18.1 (see `.nvmrc`) |
| pnpm | ^9.14.2 (`corepack enable`) |
| PostgreSQL | 15+ (or use Docker Compose) |
| Git | any recent version |

### Steps

1. **Clone the repository** (or fork it first):

   ```bash
   git clone https://github.com/<your-username>/<repo>.git
   cd <repo>
   ```

2. **Install dependencies** (pnpm workspaces):

   ```bash
   pnpm install
   ```

3. **Configure environment variables:**

   ```bash
   cp .env.example .env
   ```

   At minimum set:

   ```env
   NEXT_PUBLIC_BASE_URL=http://localhost:3000
   BETTER_AUTH_SECRET=<random 32+ character string>
   ```

   Generate a secret with:

   ```bash
   openssl rand -base64 26 | tr -dc 'a-zA-Z0-9' | head -c 32
   ```

4. **Start PostgreSQL** (skip if you already have an external database and set `POSTGRES_URL`):

   ```bash
   docker compose up -d postgres
   ```

5. **Run database migrations:**

   ```bash
   pnpm db:migrate
   ```

6. **Start the development server** (runs all workspaces in watch mode via Turborepo):

   ```bash
   pnpm dev
   ```

   The web app is now available at [http://localhost:3000](http://localhost:3000).

### Optional Dev Utilities

```bash
pnpm db:studio    # Open Drizzle Studio to browse/edit data
pnpm db:push      # Push schema changes directly (dev only)
pnpm lint         # Lint all workspaces
pnpm typecheck    # Type-check all workspaces
pnpm format:fix   # Format with Prettier
```

## Available Scripts ⚡️

| Script | Description |
| --- | --- |
| `pnpm dev` | Run all workspaces in watch mode (Turborepo) |
| `pnpm dev:next` | Run only the web app and its dependencies |
| `pnpm build` | Build all workspaces for production |
| `pnpm lint` / `pnpm lint:fix` | Run ESLint across the monorepo |
| `pnpm typecheck` | Run TypeScript type checking everywhere |
| `pnpm format` / `pnpm format:fix` | Check / apply Prettier formatting |
| `pnpm test` | Run unit tests in every workspace (excludes e2e) |
| `pnpm test:e2e` | Run Playwright end-to-end tests |
| `pnpm test:e2e:ui` | Run Playwright in interactive UI mode |
| `pnpm db:migrate` | Apply pending Drizzle migrations |
| `pnpm db:push` | Push the schema to the database (dev) |
| `pnpm db:studio` | Launch Drizzle Studio |
| `pnpm clean` | Remove all `node_modules` |
| `pnpm lint:ws` | Check workspace dependency consistency (sherif) |

## Environment Variables 🔐

Copy `.env.example` to `.env` and fill in what you need.

### Core

| Variable | Description | Required | Example |
| --- | --- | --- | --- |
| `NEXT_PUBLIC_BASE_URL` | Base URL of your installation | Yes | `http://localhost:3000` |
| `BETTER_AUTH_SECRET` | Auth encryption secret | Yes | Random 32+ char string |
| `POSTGRES_URL` | PostgreSQL connection URL | To use external DB | `postgres://user:pass@localhost:5432/db` |
| `POSTGRES_PASSWORD` | Postgres password (compose deployments) | For Docker Compose | `secret` |
| `REDIS_URL` | Redis connection URL (rate limiting) | Optional | `redis://localhost:6379` |
| `LOG_LEVEL` | Log verbosity: `debug`, `info`, `warn`, `error` | No | `info` |
| `KAN_ADMIN_API_KEY` | Admin API key for stats/admin endpoints | For admin/monitoring | `your-secret-admin-key` |
| `NEXT_API_BODY_SIZE_LIMIT` | Max API request body size (default `1mb`) | No | `50mb` |

### Email (optional)

| Variable | Description | Example |
| --- | --- | --- |
| `EMAIL_FROM` | Sender address | `"Kan <hello@mail.kan.bn>"` |
| `SMTP_HOST` | SMTP hostname | `smtp.resend.com` |
| `SMTP_PORT` | SMTP port | `465` |
| `SMTP_USER` | SMTP username | `resend` |
| `SMTP_PASSWORD` | SMTP password/token | `re_xxxx` |
| `SMTP_SECURE` | Use secure SMTP (defaults to `true`) | `true` |
| `SMTP_REJECT_UNAUTHORIZED` | Reject invalid certs (defaults to `true`) | `false` |
| `NEXT_PUBLIC_DISABLE_EMAIL` | Disable all email features | `true` |

### Authentication (optional)

| Variable | Description | Example |
| --- | --- | --- |
| `NEXT_PUBLIC_ALLOW_CREDENTIALS` | Allow email & password login | `true` |
| `NEXT_PUBLIC_DISABLE_SIGN_UP` | Disable sign ups | `false` |
| `BETTER_AUTH_TRUSTED_ORIGINS` | Allowed callback origins | `http://localhost:3000` |
| `BETTER_AUTH_ALLOWED_DOMAINS` | Allowed OIDC domains (comma-separated) | `example.com,subsidiary.com` |
| `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` | Google OAuth | `xxx.apps.googleusercontent.com` |
| `GITHUB_CLIENT_ID` / `GITHUB_CLIENT_SECRET` | GitHub OAuth | `xxx` |
| `DISCORD_CLIENT_ID` / `DISCORD_CLIENT_SECRET` | Discord OAuth | `xxx` |
| `OIDC_CLIENT_ID` / `OIDC_CLIENT_SECRET` / `OIDC_DISCOVERY_URL` | Generic OIDC | `https://auth.example.com/.well-known/openid-configuration` |

### Storage / File Uploads (optional)

| Variable | Description | Example |
| --- | --- | --- |
| `S3_REGION` | S3 region (defaults to `us-east-1`) | `WEUR` |
| `S3_ENDPOINT` | S3-compatible endpoint (blank for AWS) | `https://xxx.r2.cloudflarestorage.com` |
| `S3_ACCESS_KEY_ID` / `S3_SECRET_ACCESS_KEY` | S3 credentials | `xxx` |
| `S3_FORCE_PATH_STYLE` | Use path-style URLs | `true` |
| `S3_AVATAR_UPLOAD_LIMIT` | Max avatar size in bytes | `2097152` |
| `NEXT_PUBLIC_STORAGE_URL` | Storage service URL | `https://storage.kanbn.com` |
| `NEXT_PUBLIC_STORAGE_DOMAIN` | Storage domain | `kanbn.com` |
| `NEXT_PUBLIC_USE_VIRTUAL_HOSTED_URLS` | Virtual-hosted style URLs | `true` |
| `NEXT_PUBLIC_AVATAR_BUCKET_NAME` | Avatars bucket | `avatars` |
| `NEXT_PUBLIC_ATTACHMENTS_BUCKET_NAME` | Attachments bucket | `attachments` |

### Imports & White Labelling (optional)

| Variable | Description | Example |
| --- | --- | --- |
| `TRELLO_APP_API_KEY` | Trello app API key | `xxx` |
| `TRELLO_APP_API_SECRET` | Trello app API secret | `xxx` |
| `NEXT_PUBLIC_WHITE_LABEL_HIDE_POWERED_BY` | Hide “Powered by kan.bn” on public boards | `true` |

## Self Hosting 🐳

### One-click Deployments

The easiest way to deploy Kan is through Railway. We've partnered with Railway to maintain an official template:

<a href="https://railway.com/deploy/kan?referralCode=bZPsr2&utm_medium=integration&utm_source=template&utm_campaign=generic">
  <img src="https://railway.app/button.svg" alt="Deploy on Railway" height="40" />
</a>

### Docker Compose

Self-host with Docker Compose — it sets up PostgreSQL, runs migrations automatically, and starts the web app.

1. Create a `.env` file with your environment variables (see [Environment Variables](#environment-variables-)).

2. Use the provided [`docker-compose.yml`](./docker-compose.yml), or the following minimal configuration:

   ```yaml
   services:
     migrate:
       image: ghcr.io/kanbn/kan-migrate:latest
       container_name: kan-migrate
       networks:
         - kan-network
       environment:
         - POSTGRES_URL=${POSTGRES_URL}
       depends_on:
         postgres:
           condition: service_healthy
       restart: "no"

     web:
       image: ghcr.io/kanbn/kan:latest
       container_name: kan-web
       ports:
         - "${WEB_PORT:-3000}:3000"
       networks:
         - kan-network
       env_file:
         - .env
       environment:
         - NEXT_PUBLIC_BASE_URL=${NEXT_PUBLIC_BASE_URL}
         - BETTER_AUTH_SECRET=${BETTER_AUTH_SECRET}
         - POSTGRES_URL=${POSTGRES_URL}
         - NEXT_PUBLIC_ALLOW_CREDENTIALS=true
       depends_on:
         migrate:
           condition: service_completed_successfully
       restart: unless-stopped

     postgres:
       image: postgres:15
       container_name: kan-db
       environment:
         - POSTGRES_DB=kan_db
         - POSTGRES_USER=kan
         - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
       ports:
         - 5432:5432
       volumes:
         - kan_postgres_data:/var/lib/postgresql/data
       healthcheck:
         test: ["CMD-SHELL", "pg_isready -U kan -d kan_db"]
         interval: 5s
         timeout: 5s
         retries: 10
       restart: unless-stopped
       networks:
         - kan-network

   networks:
     kan-network:

   volumes:
     kan_postgres_data:
   ```

3. Start everything in detached mode:

   ```bash
   docker compose up -d
   ```

The `migrate` service runs database migrations before the web service starts. The app is then available at `http://localhost:3000` (or your `WEB_PORT`).

**Managing containers:**

| Task | Command |
| --- | --- |
| Stop | `docker compose down` |
| View all logs | `docker compose logs -f` |
| View one service's logs | `docker compose logs -f web` |
| Restart | `docker compose restart` |
| Rebuild after code changes | `docker compose up -d --build` |

For the complete configuration (including optional features such as Redis), see [`docker-compose.yml`](./docker-compose.yml). A cloud-specific variant lives in [`cloud/docker-compose.yml`](./cloud/docker-compose.yml).

### Building Images Locally

If you prefer building from source instead of pulling the published images:

```bash
docker compose build
docker compose up -d
```

## Testing 🧪

```bash
pnpm test             # Unit tests across all workspaces
pnpm test:e2e         # Playwright E2E (headless)
pnpm test:e2e:ui      # Playwright interactive UI mode
pnpm typecheck        # TypeScript across the monorepo
pnpm lint             # ESLint across the monorepo
```

E2E reports are written to `packages/e2e/playwright-report/` (git-ignored).

## MCP Server (AI Control) 🤖

Kan ships with a [Model Context Protocol](https://modelcontextprotocol.io) (MCP) server that lets any MCP-compatible AI client (Claude Desktop, Codex, Cursor, GitHub Copilot, and others) read and control your Kan instance using natural language.

Run it with `npx` — no clone or global install required:

```bash
npx -y @kan/mcp
```

Configure it with two environment variables:

- `KAN_BASE_URL` — your Kan instance URL
- `KAN_API_TOKEN` — an API token from **Settings → API Keys**

Then point your client's MCP config at the `npx -y @kan/mcp` command. See the [MCP Server docs](https://docs.kan.bn/integrations/mcp-server) for per-client configuration, example prompts, the full tool reference, and troubleshooting.

## Contributing 🤝

Contributions are welcome! Before submitting a pull request:

1. Read the [contribution guidelines](CONTRIBUTING.md) and [`AGENTS.md`](AGENTS.md).
2. Follow the existing code style — run `pnpm lint`, `pnpm typecheck`, and `pnpm format:fix`.
3. Keep commits focused and use [Conventional Commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `refactor:`, `docs:` …).
4. Add or update translations for user-facing strings (`pnpm lingui:extract`).
5. Test database changes against a development database first, and never modify existing migrations — generate new ones:

   ```bash
   cd packages/db && pnpm drizzle-kit generate --name "MigrationName"
   pnpm db:migrate
   ```

## License 📝

Kan is licensed under the [AGPLv3 license](LICENSE).

## Contact 📧

For support or to get in touch, please email [henry@kan.bn](mailto:henry@kan.bn) or join the [Discord server](https://discord.gg/e6ejRb6CmT).

---

## About this fork

Built by **Girish Lade** — [ladestack.in](https://ladestack.in)
