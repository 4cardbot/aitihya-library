---
layout: plain
title: "Aitihya / Pramāṇa: database implementation"
permalink: /research/aitihya-database-implementation.html
---

# Aitihya / Pramāṇa: database implementation

**Status:** implemented in code, applied to the development Supabase project and verified with five demo product records.
**Mode:** demo by default; connected mode when `PRAMANA_MODE=connected` and `DATABASE_URL` are present.

## What is now in the repository

- `src/db/schema.ts`: typed Drizzle schema;
- `src/db/client.ts`: lazy server-side Postgres connection;
- `supabase/migrations/0000_sudden_dreadnoughts.sql`: tables, indexes, foreign keys, triggers and RLS policies;
- `scripts/migrate.ts`: applies migrations with Drizzle;
- `scripts/seed.ts`: idempotent demo record seed;
- `src/lib/pramana-repository.ts`: connected-mode product lookup, scan persistence and trade-inquiry persistence;
- `src/lib/supabase/`: browser/server Auth clients and Next.js 16 session proxy;
- `src/lib/auth.ts`: staff-role authorization for the private studio;
- `src/app/studio/page.tsx`: authenticated registry dashboard with product draft/content/publication and enquiry status controls;
- `src/db/schema.test.ts`: schema inventory test.

## Table inventory

The implementation contains 33 tables. The original 26 Pramāṇa entities are present, with relationship tables, journal content and a commercial intake table added where the application needs them:

| Area | Tables |
|---|---|
| Editorial/product identity | `collections`, `designs`, `design_versions`, `products`, `editions` |
| Editorial journal | `journal_entries` |
| Material/technique/place | `materials`, `material_batches`, `techniques`, `regions`, `ateliers`, `makers`, `product_material_batches`, `product_techniques`, `product_attributions`, `community_protocols` |
| Evidence/consent | `claims`, `claim_evidence`, `consents`, `maker_attribution_permissions`, `quality_checks`, `condition_reports`, `certificates` |
| Media/tag layer | `media_assets`, `media_rights`, `tag_links`, `scan_events` |
| Commercial/continuity | `trade_inquiries`, `orders`, `ownership_records`, `service_records` |
| Governance | `user_profiles`, `audit_events` |

## Data protection choices

- Exact GPS is not a column and is not accepted by the QR API.
- Public tokens are separate from internal UUIDs and human-readable product codes.
- Public product reads require `status = 'published'`.
- Public media reads require both `status = 'published'` and `visibility = 'public'`.
- Registry writes, scan persistence and trade intake occur server-side.
- Staff access is controlled through `user_profiles.role` and RLS policies.
- Owner records are restricted to the matching user or Pramāṇa staff.
- Certificate and audit rows are not exposed to anonymous clients.

## Run it

```bash
cp .env.example .env.local
# Set DATABASE_URL and PRAMANA_MODE=connected in .env.local
npm run db:migrate
npm run db:seed
```

For production, use separate Supabase projects and connection strings for development, staging and production. Apply the migration in staging first, inspect the RLS behavior with anonymous/authenticated/staff test users, then apply it to production.

## Still required before production

- Create separate staging and production Supabase projects and provide their connection strings through server environment variables.
- Replace the five demo records with approved product, maker, consent and media records.
- Add the first production Supabase Auth users and test each role in staging.
- Expand the studio from core product/intake controls into full Pramāṇa CRUD and approval workflow screens for evidence, makers, media and certificates.
- Connect media uploads to R2/Cloudflare Stream.
- Add certificate PDF generation and version/revocation UI.
- Add HubSpot/Odoo webhooks and reconciliation jobs.
- Add rate limiting, structured audit logging and a restore test against the production backup policy.

