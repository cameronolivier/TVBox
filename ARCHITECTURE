🧱 Architecture Document

1. System Overview

Architecture type: Polyglot monorepo (TypeScript-first, shared contracts)

Components

Layer	Tech Stack	Responsibility
Server API	TypeScript + Fastify + Drizzle ORM (Postgres)	Schedule generation, API endpoints, media serving
Database	Postgres	Assets, folders, channels, schedule items, ad inventory
Admin UI	Next.js + tRPC + Tailwind	Manage channels, library, EPG
Clients	Android (Kotlin + ExoPlayer), Xbox (UWP + C#)	Consume /now API, display playback
Shared contracts	Zod + zod-to-openapi	Typed schemas across stack
Storage	Local FS or mounted NAS	Original media + transcoded assets
Workers	BullMQ / cron tasks	Periodic rescans + schedule rebuilds


⸻

2. Architectural Principles
	•	Determinism — Schedules are reproducible given a seed; identical inputs yield identical EPGs.
	•	Wall-clock truth — Server’s UTC time defines “now.”
	•	Stateless clients — Clients only query /now; no local timeline.
	•	Schema-driven contracts — Shared Zod schemas generate runtime + compile-time validation.
	•	Loose coupling — Separate apps share types via package, but can deploy independently.
	•	Pure TypeScript stack — Drizzle ORM for DB migrations + TS-native typing.
	•	Infrastructure-as-code — Docker Compose → Fly.io or bare metal deployment.

⸻

3. Tech Stack

Layer	Tech	Notes
Server	Fastify + TypeScript + Drizzle (Postgres)	Minimal deps, strong perf
Validation	Zod	Contract source of truth
ORM	Drizzle ORM	SQL-first + typesafe migrations
Job queue	BullMQ	For scans and schedule rebuilds
Admin UI	Next.js (App Router) + tRPC + Tailwind + shadcn/ui	Same codebase as server
Android	Kotlin + ExoPlayer + Compose for TV	Linear playback
Xbox	C# + UWP + MediaPlayerElement	Live playout
Infra	Docker Compose + Nginx	Serve API + media
Auth	None (local admin only)	Add later if public
Media tools	ffmpeg + ffprobe	Scan + pre-render slates


⸻

4. API Structure (REST + OpenAPI)

Method	Path	Description
GET	/channels	List all channels
POST	/channels	Create channel
GET	/channels/:slug/now	Return current item + offset
GET	/channels/:slug/epg?from&to	Get schedule window
POST	/channels/:slug/rebuild	Force schedule regeneration
GET	/library/folders	List library folders
POST	/library/folders/:id/rescan	Re-scan for new assets
GET	/ads	List ad inventory
POST	/ads/import	Import PD ads from Archive.org


⸻

5. Database Schema (Drizzle ORM)

import { pgTable, text, boolean, integer, timestamp, jsonb } from 'drizzle-orm/pg-core'

export const mediaAssets = pgTable('media_assets', {
	id: text('id').primaryKey(),
	path: text('path').notNull(),
	folderId: text('folder_id').notNull(),
	filename: text('filename').notNull(),
	durationMs: integer('duration_ms').notNull(),
	width: integer('width'),
	height: integer('height'),
	hash: text('hash').notNull(),
	addedAt: timestamp('added_at').defaultNow(),
	metadata: jsonb('metadata'),
})

export const channels = pgTable('channels', {
	id: text('id').primaryKey(),
	name: text('name').notNull(),
	slug: text('slug').notNull().unique(),
	timezone: text('timezone').default('UTC'),
	isLive: boolean('is_live').default(true),
	policy: jsonb('policy').notNull(),
	createdAt: timestamp('created_at').defaultNow(),
	updatedAt: timestamp('updated_at').defaultNow(),
})

export const scheduleItems = pgTable('schedule_items', {
	id: text('id').primaryKey(),
	channelId: text('channel_id').notNull(),
	assetId: text('asset_id'),
	type: text('type').notNull(), // 'asset' | 'slate' | 'ad'
	startUtc: timestamp('start_utc').notNull(),
	endUtc: timestamp('end_utc').notNull(),
	title: text('title'),
	meta: jsonb('meta'),
})

export const adAssets = pgTable('ad_assets', {
	id: text('id').primaryKey(),
	title: text('title'),
	sourceUrl: text('source_url'),
	localPath: text('local_path'),
	license: text('license'),
	durationMs: integer('duration_ms'),
	eraTag: text('era_tag'),
})


⸻

6. Zod Schemas (Shared Contracts)

import { z } from 'zod'

export const ZScheduleItem = z.object({
	id: z.string(),
	channelId: z.string(),
	type: z.enum(['asset', 'slate', 'ad']),
	startUtc: z.string().datetime(),
	endUtc: z.string().datetime(),
	title: z.string().optional(),
	meta: z.record(z.unknown()).optional(),
})

export const ZNowResponse = z.object({
	item: ZScheduleItem,
	offsetMs: z.number().int().nonnegative(),
	assetUrl: z.string().url(),
	serverTimeUtc: z.string().datetime(),
})

export const ZAutoSchedulePolicy = z.object({
	ordering: z.enum(['shuffle', 'filenameAsc', 'recentFirst']),
	alignBlocksMinutes: z.number().optional(),
	insertEvery: z
		.object({ minutes: z.number().optional(), files: z.number().optional(), type: z.string() })
		.optional(),
	allowRepeatWithinHours: z.number(),
	gapFill: z.enum(['loop', 'slate']),
	ads: z
		.object({
			enable: z.boolean(),
			insertEveryMinutes: z.number(),
			breakDurationSeconds: z.number(),
			eraBias: z.array(z.object({ era: z.enum(['80s', '90s']), weight: z.number() })).optional(),
		})
		.optional(),
	blackouts: z
		.array(
			z.union([
				z.object({ kind: z.literal('daily'), startLocal: z.string(), stopLocal: z.string() }),
				z.object({ kind: z.literal('once'), startUtc: z.string(), stopUtc: z.string() }),
			])
		)
		.optional(),
})


⸻

7. Testing Strategy

Testing levels

Level	Tool	Purpose
Unit	Vitest	Validate schedule logic, offset math
Integration	Supertest + Fastify instance	Validate API responses and time offsets
Contract	Zod inference tests	Ensure shared schemas match OpenAPI
E2E	Playwright (Admin UI)	Basic CRUD flow for channels
Regression	Snapshot	EPG determinism given same seed

Must-test areas

Component	Tests	Why critical
Scheduler	Deterministic EPG generation, repeat prevention, alignment logic	Prevent drift and duplicates
Offset computation	/now returns correct offset for current UTC time	Core playback correctness
Blackouts	Correctly produces slate for off-air windows	Prevent missing media
Ad insertion	Ad break timing + total duration matches target	UX & timing fidelity
API schema	Type-safe validation matches OpenAPI	Cross-client compatibility
Client boundary transitions	When near end of file, next /now returns seamless transition	No playback gaps


⸻

8. Automated Testing Focus
	•	Property-based testing for schedule generation (QuickCheck-style: total coverage of 24h always = expected duration).
	•	Mock clock tests simulating UTC changes.
	•	Contract drift tests — ensure Android/Xbox codegen clients pass type validation.
	•	ffprobe scanner mocks — assert asset metadata parsing.
	•	Zod parsing — fuzz test invalid API payloads.

⸻

9. Deployment / DevOps

Env	Tool
Local	Docker Compose: Postgres, Fastify API, Next.js Admin
CI	GitHub Actions + Turbo caching
Prod	Fly.io or Railway (stateless container)
Media storage	Bind mount or S3
Cron jobs	BullMQ or systemd timer for rebuildSchedules() daily


⸻

10. Key Architectural Decisions

Decision	Rationale
Monorepo (pnpm + Turborepo)	Shared contracts, atomic CI
Drizzle ORM	Simpler migration/versioning than Prisma; pure TypeScript
Zod-first contracts	One source of truth for API + clients
Fastify	Superior perf + plugin ecosystem; perfect for low-latency /now
Static media serving	Avoid CPU-intensive live concat
Weighted deterministic scheduling	Predictable results; debuggable
Wall-clock source = server UTC	Single source of truth for “now”
Cross-platform clients	Minimal logic, same API contract
Public-domain ad ingestion	Legal & reproducible
Simple auth boundary (local admin only)	MVP scope control


⸻

11. Future Enhancements
	•	Server-side stitched HLS for seamless transitions
	•	Cloud storage + signed URLs
	•	AI-generated bumpers / slates
	•	Program guide overlay (EPG grid UI on TV)
	•	Multi-user channels with remote sync
	•	“Now/Next” overlay personalization per channel

⸻

✅ Summary

This architecture achieves:
	•	Full determinism: wall-clock-based scheduling guarantees sync across clients.
	•	Schema alignment: one Zod contract drives all apps.
	•	Drizzle as shared ORM: efficient, consistent Postgres layer.
	•	Extensibility: easily add new media sources, ad policies, or client platforms.
	•	Safety: rights-clean ad ingestion and test pattern generation.
