---
layout: plain
title: "Aitihya / Pramāṇa: software infrastructure blueprint"
permalink: /research/aitihya-software-infrastructure.html
---

# Aitihya / Pramāṇa: software infrastructure blueprint

**Research cut-off:** 26 September 2026  
**Scope:** launch through the first 24 months, with a migration path suitable for a 36–48 month strategic sale.  
**Primary principle:** use managed software for ordinary business functions; build proprietary software only around Pramāṇa, product identity, provenance, media rights and the designer workflow.

## 1. The architecture decision

Build a **modular monolith with managed services**, not a collection of microservices.

The first version should be fast, maintainable by a small team and portable to an acquirer. Aitihya does not need a mobile app, blockchain, custom ERP or a large engineering department at launch.

The proprietary core is:

- the Pramāṇa Product ID;
- the provenance and evidence model;
- the QR/NFC Story Page;
- the media and consent record;
- the designer sample/specification workflow;
- the connection between product, production, customer, repair and ownership history.

Everything else should be bought or configured until there is a clear reason to build it.

## 2. Recommended stack

### 2.1 Launch stack

| Function | Recommendation | Why |
|---|---|---|
| Source control | GitHub private repository | Code ownership, review, audit trail and future acquirer handover |
| CI/CD | GitHub Actions | Automated linting, tests, security checks and deployment workflows |
| Frontend and application | Next.js, TypeScript, App Router | One codebase for editorial website, product pages, Story Pages and trade portal |
| UI system | Tailwind CSS plus a small Aitihya design-token layer | Fast, consistent implementation without locking brand decisions inside a theme |
| Hosting | Vercel for the web application, behind Cloudflare | Preview deployments, global delivery and a straightforward Next.js workflow |
| DNS/WAF/CDN | Cloudflare DNS, WAF, Bot Management and Turnstile where appropriate | Domain control, rate limiting, redirects, security and caching |
| Editorial CMS | Sanity | Structured collections, materials, stories, films, designers and modular editorial content |
| Core database | PostgreSQL through Supabase | Managed relational database, authentication, backups, row-level access controls and a Mumbai region option |
| ORM and validation | Drizzle ORM plus Zod | Typed database access and validated inputs at API boundaries |
| Authentication | Supabase Auth | Invite-only trade access, owner registration, magic links and admin MFA support |
| Media masters | Cloudflare R2 or equivalent S3-compatible object storage | Durable source files, consent documents, certificates and exportable media archive |
| Video delivery | Cloudflare Stream | Upload, encode, deliver and play documentaries without operating a video pipeline |
| CRM | HubSpot CRM | Designer accounts, pipeline, tasks, email history, sample loans and private-client follow-up |
| Operations and inventory | Odoo Manufacturing/Inventory/Quality, configured conservatively | Bills of materials, lot/serial tracking, production orders, QC and replenishment |
| Payments | Indian gateway plus international gateway after finance approval | Use the provider that is eligible for the entity, currencies, export documentation and settlement structure |
| Product analytics | PostHog with privacy controls | Funnels for Story Pages, sample requests, trade portal and owner journeys |
| Error monitoring | Sentry | Frontend/backend errors, releases and performance issues |
| Uptime/logging | Better Stack or equivalent | Uptime checks, structured logs and incident notifications |
| Appointments | CRM-native meeting booking initially | Keeps appointment history attached to the account and avoids another disconnected system |
| Documentation | Notion or an equivalent internal knowledge base, with signed records stored separately | Operating playbooks, launch checklists and team knowledge |

This recommendation is based on current official documentation: Next.js supports Node.js, Docker and platform adapters; Vercel provides Git-based deployments and preview environments; Supabase provides managed Postgres, backups and row-level security; Sanity provides structured schemas and Next.js integrations; Cloudflare Stream manages video storage, encoding and delivery; GitHub Actions supports build/test/deploy workflows. [Next.js deployment](https://nextjs.org/docs/app/getting-started/deploying), [Vercel](https://vercel.com/docs/frameworks/full-stack/nextjs), [Supabase regions](https://supabase.com/docs/guides/platform/regions), [Sanity APIs](https://www.sanity.io/docs/apis-and-sdks), [Cloudflare Stream](https://www.cloudflare.com/products/stream/), [GitHub Actions](https://github.com/features/actions)

### 2.2 What not to use at launch

- No Kubernetes.
- No microservice fleet.
- No custom mobile app.
- No blockchain or NFT layer.
- No live GPS tracking.
- No custom payment storage.
- No self-hosted video transcoding.
- No CRM built in spreadsheets.
- No public API until the internal data model is stable.
- No large ERP customisation before the first paid production orders.

## 3. Where each system should be deployed

### 3.1 Production topology

```text
Customer / designer / QR scan
            ↓
Cloudflare DNS + WAF + CDN
            ↓
Vercel: Next.js website, Story Pages, trade portal and API routes
            ├── Sanity: editorial content and structured media references
            ├── Supabase Mumbai: Pramāṇa database, auth and application data
            ├── Cloudflare Stream: video encoding and playback
            ├── Cloudflare R2: masters, certificates and private media
            ├── HubSpot: CRM and trade pipeline
            └── Odoo: operations, orders, inventory and quality
```

### 3.2 Data region

Use the **Mumbai (`ap-south-1`) region** for the primary Supabase production project if the selected plan and legal review support it. Supabase lists Mumbai as an available specific AWS region and notes that the chosen region controls where primary project data is stored, but regional location alone is not proof of regulatory compliance. [Supabase regions](https://supabase.com/docs/guides/platform/regions)

Create separate projects for:

- `pramana-dev`;
- `pramana-staging`;
- `pramana-production`.

Never use production customer, owner, maker or consent data in development.

If a future acquirer requires AWS-native infrastructure, the Pramāṇa API and PostgreSQL database can move to AWS RDS/ECS or Lambda without rebuilding the frontend, provided the team keeps the application behind a clean repository interface and exports regular database backups.

### 3.3 Environments

| Environment | Purpose | Data |
|---|---|---|
| Local | Developer work | Synthetic or anonymised fixtures only |
| Preview | Pull-request review and content QA | Sanitised staging data |
| Staging | Release testing, integrations and UAT | Test products, tags and test accounts |
| Production | Live website, registry and trade workflows | Real data with backups and access controls |

Vercel supports local, preview and production deployment patterns connected to Git workflows. Use preview deployments for every meaningful change to product pages, Story Pages and trade workflows. [Vercel deployments](https://vercel.com/docs/deployments/overview)

## 4. Application boundaries and data ownership

Do not allow every platform to become a partial source of truth. Assign ownership explicitly:

| Data | System of record | Other systems receive |
|---|---|---|
| Product identity, edition, provenance and evidence | Pramāṇa | Read-only references in website, Odoo and certificates |
| Editorial story, collection and journal content | Sanity | Published references in website, email and social assets |
| Designer/private-client relationship | HubSpot | Customer/account IDs in orders and Pramāṇa |
| Order, invoice, production, inventory and QC status | Odoo | Order/product status in Pramāṇa and CRM |
| Video masters, subtitles, consent and usage rights | R2 + media metadata in Pramāṇa/Sanity | Stream playback IDs and public links |
| Payment transaction | Approved payment provider and finance system | Payment status and reference only |
| Scan and funnel events | PostHog, with essential events mirrored in Pramāṇa where needed | Aggregated reporting |

### 4.1 Canonical identifiers

Every product needs:

- internal UUID/ULID: never exposed publicly;
- human-readable product ID, for example `AT-PNL-2026-0007`;
- edition number if applicable;
- batch/lot reference;
- QR token;
- NFC tag UID or secure tag reference;
- Pramāṇa certificate ID;
- Odoo product/lot/serial reference;
- optional owner registration ID.

The public QR should contain an opaque token or dynamic short URL, not an incrementing database ID. This prevents easy enumeration of products.

## 5. Pramāṇa data model

### 5.1 Core entities

```text
Product
 ├── Design version
 ├── Edition / serial
 ├── Material batch
 ├── Technique
 ├── Region / atelier / maker attribution
 ├── Evidence and consent records
 ├── QC and condition reports
 ├── Certificate(s)
 ├── Media assets and documentary
 ├── Tag links: QR / NFC
 ├── Order and dispatch reference
 ├── Owner / transfer history
 └── Repair / conservation history
```

### 5.2 Minimum database tables

- `products`
- `designs`
- `design_versions`
- `editions`
- `materials`
- `material_batches`
- `techniques`
- `regions`
- `ateliers`
- `makers`
- `maker_attribution_permissions`
- `community_protocols`
- `claims`
- `claim_evidence`
- `consents`
- `quality_checks`
- `condition_reports`
- `certificates`
- `media_assets`
- `media_rights`
- `tag_links`
- `scan_events`
- `orders`
- `ownership_records`
- `service_records`
- `audit_events`

The implementation adds the relationship tables `product_material_batches`, `product_techniques` and `product_attributions`, plus the commercial `trade_inquiries` table and `user_profiles` needed by the application. The complete implementation inventory is in [`aitihya-database-implementation.md`](./aitihya-database-implementation.html).

### 5.3 Certificate workflow

```text
Draft product
   ↓
Evidence and consent attached
   ↓
QC completed
   ↓
Certificate scope selected: P1–P5
   ↓
Reviewer approves record
   ↓
Certificate PDF/hash generated
   ↓
QR/NFC provisioned
   ↓
Product released for sale
```

Certificates must be versioned. If a material record, care instruction or technical result changes, do not silently overwrite the old record. Create a new version and preserve the previous audit event.

The class should describe the evidence attached to the record, not imply that Pramāṇa is an independent statutory certifier:

| Class | Use at launch |
|---|---|
| P1 — identity record | Product ID, design, edition, maker/atelier attribution and approved public story |
| P2 — material record | P1 plus material, batch, technique, care and supporting evidence |
| P3 — verified production record | P2 plus production/QC record, dimensions, condition and release approval |
| P4 — technical record | P3 plus commissioned acoustic, fire, installation or other relevant test documents |
| P5 — bespoke/private record | P3/P4 as relevant plus restricted client, installation, ownership and service records |

Until independent governance is established, public language should say “Pramāṇa-registered”, “Pramāṇa-documented” or “Pramāṇa-verified” according to the evidence class. Do not use “independently certified” without an independent reviewer and defined standard.

### 5.4 Audit trail

For every sensitive change, store:

- actor;
- timestamp;
- old value and new value or a change summary;
- reason;
- evidence attached;
- approval status;
- source system reference.

A conventional append-only audit log with database backups is sufficient initially. A cryptographic hash of the certificate/evidence bundle can be stored for tamper evidence. A public blockchain is unnecessary.

## 6. QR, NFC and Story Page implementation

### 6.1 Dynamic QR flow

```text
QR printed on product
        ↓
https://your-domain.example/p/opaque-token
        ↓
Resolve token and check product status
        ↓
Render public Story Page
        ↓
Load documentary, certificate, care and provenance data
        ↓
Record privacy-safe scan event
```

The URL should remain stable even if the film, owner status or care document changes.

### 6.2 NFC hierarchy

- Core product: QR on sewn label or certificate card.
- High-value core: QR plus passive NFC.
- Numbered edition: secure NFC with cryptographic authentication plus QR fallback.
- Bespoke commission: secure NFC on the reverse mounting or archival certificate, not where washing or friction will damage it.

For secure editions, use a tag with authenticated reads and protected data. NXP’s NTAG 424 DNA is one current option supporting AES-128 authentication, protected data and tamper-aware variants. [NXP NTAG 424 DNA](https://www.nxp.com/products/rfid-nfc/nfc-hf/ntag-for-tags-and-labels/ntag-424-dna-424-dna-tagtamper-advanced-security-and-privacy-for-trusted-iot-applications%3ANTAG424DNA)

### 6.3 Scan-event policy

Record only what is useful:

- product/token;
- time;
- approximate country or region where available and consented;
- device category;
- Story Page version;
- whether the owner or public view was used.

Do not store exact GPS from a customer’s phone by default. Never expose a client’s name, address, purchase value or room photographs on a public page.

## 7. Cinematic video infrastructure

### 7.1 Media pipeline

```text
Camera master
   ↓
R2 master archive + media-rights record
   ↓
Cloudflare Stream upload/encoding
   ↓
Captions, transcript, thumbnail and alt text
   ↓
Pramāṇa Story Page
   ↓
Social cutdowns and trade portal assets
```

Use Cloudflare Stream for the access copy and playback, while retaining high-resolution masters in R2 or an equivalent object store. Cloudflare Stream provides upload, encoding, delivery and playback through one managed pipeline. [Cloudflare Stream](https://www.cloudflare.com/products/stream/)

### 7.2 Media metadata

Every media asset needs:

- asset ID;
- linked product/design/collection;
- title and description;
- language and subtitle files;
- transcript;
- people and locations shown;
- consent and release records;
- music and archive licences;
- publication status;
- expiration/review date;
- public/trade/private visibility;
- master, access and social-cut versions.

Use signed playback URLs for private owner films, private commissions and unpublished material.

## 8. Website and content architecture

### 8.1 Public route map

```text
/
/collections
/collections/[slug]
/products/[slug]
/p/[opaque-product-token]
/materials/[slug]
/techniques/[slug]
/rooms/[slug]
/journal/[slug]
/pramana
/trade
/trade/login
/trade/library
/appointments
/care-and-repair
/contact
```

### 8.2 Product page requirements

Every product page must include:

- image set and short film;
- dimensions and material composition;
- made-to-order/stock status;
- expected lead time;
- care instructions;
- price or enquiry state;
- trade suitability;
- certificate class;
- sample request CTA;
- private appointment CTA;
- related room and material stories.

### 8.3 CMS content types

Sanity should model meaning rather than hard-code one presentation. Use schemas for:

- collection;
- product;
- design version;
- material;
- technique;
- region;
- maker/atelier profile;
- room/project;
- documentary;
- field note;
- designer profile;
- certificate explanation;
- care guide;
- press asset.

Sanity’s current documentation supports schema-based structured content and Next.js integrations, which suits a brand that will need the same stories across web, product pages, trade materials and social output. [Sanity APIs and SDKs](https://www.sanity.io/docs/apis-and-sdks), [Sanity structured content](https://www.sanity.io/docs/developer-guides/how-to-use-structured-content-for-page-building)

## 9. CRM, trade and private-client workflows

### 9.1 CRM pipelines

Create separate pipelines:

1. **Designer account:** identified → introduced → meeting → sample loan → active brief → quote → deposit → repeat.
2. **Private client:** enquiry → qualification → appointment → edition shortlist → reservation → payment → delivery → care.
3. **Bespoke commission:** brief → paid development → concept → sample → approval → deposit → production → installation → handover.
4. **Hospitality/technical:** opportunity → technical brief → test/specification → quote → pilot → project order.

HubSpot’s current CRM documentation supports contact history, deals, tasks, pipeline management, reporting and permissions. [HubSpot CRM](https://www.hubspot.com/products/crm)

### 9.2 Trade portal permissions

- Public visitor: collection and editorial content.
- Approved designer: trade prices, samples, specifications and lead times.
- Project account: room-specific files, quotes and approvals.
- Private owner: care, ownership and service records.
- Internal team: operational and financial data.
- External reviewer: limited Pramāṇa evidence and certificate workflow.

## 10. Operations, inventory and quality

Use Odoo for the operational layer, not for the full Pramāṇa evidence model.

Configure:

- products and variants;
- bills of materials where applicable;
- purchase orders and workshop partners;
- lot and serial numbers;
- production stages;
- quality-control points;
- image-based inspection;
- repair/rework status;
- dispatch and returns;
- minimum stock and make-to-order rules.

Odoo’s current documentation supports quality checks, control points, photos, measured tolerances, lot/serial tracking and make-to-order workflows. [Odoo Quality](https://www.odoo.com/documentation/18.0/applications/inventory_and_mrp/quality.html), [Odoo quality checks](https://www.odoo.com/documentation/18.0/applications/inventory_and_mrp/quality/quality_management/quality_checks.html), [Odoo lots and serial numbers](https://www.odoo.com/documentation/18.0/applications/inventory_and_mrp/inventory/product_management/product_tracking/lots.html)

Do not duplicate the full provenance record in Odoo. Store a stable Pramāṇa Product ID and certificate reference there.

## 11. Analytics and measurement

### 11.1 Events to implement

Public website:

- `collection_viewed`;
- `product_viewed`;
- `film_started`;
- `film_25_percent`;
- `film_completed`;
- `material_library_requested`;
- `appointment_requested`;
- `trade_application_started`;
- `trade_application_submitted`;
- `quote_requested`.

Pramāṇa:

- `qr_scanned`;
- `nfc_authenticated`;
- `certificate_viewed`;
- `owner_registration_started`;
- `owner_registration_completed`;
- `care_downloaded`;
- `repair_requested`;
- `ownership_transfer_requested`.

Sales/operations:

- `sample_dispatched`;
- `brief_created`;
- `quote_created`;
- `deposit_received`;
- `production_started`;
- `qc_failed`;
- `qc_passed`;
- `dispatch_completed`;
- `repeat_order_created`.

PostHog supports event-based trends, funnels, retention and paths; instrument only events that affect decisions, and do not send sensitive client or maker data into analytics. [PostHog product analytics](https://posthog.com/docs/product-analytics)

### 11.2 Executive dashboard

Weekly dashboard:

- qualified accounts and active briefs;
- sample-to-brief rate;
- quote-to-deposit rate;
- deposits and days to cash;
- product contribution margin;
- production lead-time adherence;
- QC failure/rework rate;
- QR scans and Story Page completion;
- repair/service requests;
- repeat and referral revenue.

## 12. Security, privacy and business continuity

### 12.1 Minimum controls

- MFA for all admin and finance accounts.
- Passwordless or magic-link login for invited trade users, with expiry.
- Role-based access in Supabase, CRM, CMS and Odoo.
- Separate public, trade, internal and owner data.
- Signed URLs for private media and documents.
- Rate limiting and bot protection on QR and login endpoints.
- No payment-card storage in Aitihya systems.
- No secrets or API keys in the repository.
- Dependency and container vulnerability scanning.
- Audit logs for certificates, ownership and evidence changes.
- Quarterly access review.
- Vendor data-processing and media-rights review.

### 12.2 Backups and recovery

Target:

- database point-in-time recovery on the paid production plan;
- daily encrypted database export to a separate storage location;
- weekly restore test for the database;
- versioned media masters;
- monthly export of Pramāṇa records and certificate metadata;
- documented recovery contacts and runbook.

Planning targets:

- RPO: 24 hours for ordinary content; near-zero loss for certificate and ownership changes;
- RTO: 4 hours for public website; 8 hours for internal trade tools.

These are operational targets to agree with the team, not vendor guarantees.

### 12.3 Privacy boundaries

Publicly expose only what has been approved. Exact artisan coordinates, private community knowledge, client addresses, purchase values and private-room images belong behind restricted access or should not be stored at all.

Obtain legal review for Indian, UK, US and UAE privacy, consumer, marketing, documentary-consent, data-transfer and retention requirements before collecting sensitive information.

## 13. Engineering workflow

### 13.1 Repository structure

```text
aitihya-platform/
  apps/
    web/              # Next.js public site, Story Pages and trade portal
    pramana-admin/    # internal registry/admin UI, if split becomes useful
  packages/
    design-system/    # tokens and reusable UI
    domain/           # product, certificate and workflow types
    db/               # migrations and Drizzle schema
    integrations/     # Sanity, HubSpot, Odoo, Stream and payments
    analytics/        # event names and tracking helpers
  tests/
    unit/
    integration/
    e2e/
```

Start with one Next.js application and one shared domain package. Split the admin UI only when permissions or team size justify it.

### 13.2 Pull-request gates

Every production change should pass:

- TypeScript typecheck;
- ESLint and formatting;
- unit tests for registry and certificate logic;
- integration tests for QR resolution and webhook handling;
- Playwright tests for Story Pages, trade login and owner flows;
- accessibility checks;
- dependency/security scan;
- preview deployment review;
- migration review for database changes.

GitHub Actions can run these workflows on pull requests and deploy approved changes. [GitHub Actions](https://docs.github.com/en/actions/get-started/understand-github-actions)

### 13.3 Release rules

- `main` deploys to production only after review.
- Database migrations are forward-compatible and reversible where possible.
- Certificates and audit records are never hard-deleted through the UI.
- Feature flags control new owner, trade and payment workflows.
- Every release has a rollback plan.
- No production hotfix bypasses an incident note.

## 14. Implementation roadmap

### Weeks 1–2: foundation

- Buy/secure domains and social handles.
- Create GitHub organisation and private repository.
- Set up Cloudflare, Vercel, Supabase Mumbai, Sanity and Stream accounts.
- Create dev/staging/production projects.
- Define Product ID and Pramāṇa schema.
- Define roles, permissions and data classification.

### Weeks 3–6: MVP

- Build the public website shell.
- Build collection/product pages.
- Build `/p/[token]` Story Page.
- Implement QR generation and scan logging.
- Create Pramāṇa admin for products, evidence, media and certificates.
- Add documentary playback, captions and transcripts.
- Set up HubSpot pipeline and trade enquiry form.

### Weeks 7–10: operational connection

- Connect product records to Odoo references.
- Add sample requests, appointments and quote requests.
- Add certificate PDF generation and versioning.
- Add owner registration and care page.
- Implement analytics events and dashboards.
- Run security, accessibility and restore tests.

### Months 4–6: launch hardening

- Add secure NFC for numbered editions.
- Add private trade portal.
- Add media-rights and consent workflow.
- Add repair/conservation requests.
- Add production and QC dashboards.
- Add email and private-client automations.

### Months 6–24: selective expansion

- Add acoustic/technical product records and test documents.
- Add owner transfer and authenticated resale support.
- Add multilingual subtitles and market-specific content.
- Add external Pramāṇa review workflow if volume justifies it.
- Export data and architecture documentation for future diligence.

## 15. Team and ownership

### First six months

- Product/digital lead: owns roadmap, data model and vendor coordination.
- Full-stack engineer: owns web, Pramāṇa and integrations.
- Brand/editorial producer: owns films, media archive and publishing.
- CRM/trade operations manager: owns accounts, sample loans and follow-up.
- Fractional security/privacy adviser: reviews access, consent and contracts.

### Ownership matrix

| Area | Accountable owner |
|---|---|
| Product identity and certificate truth | Pramāṇa/product lead |
| Brand/editorial truth | Creative lead and Anjan |
| Customer relationship truth | Commercial/trade lead |
| Financial/order truth | Finance/operations lead |
| Infrastructure/security | Digital lead plus external adviser |

## 16. Launch acceptance checklist

Do not publicly launch the full product system until:

- the website works on mobile and slow connections;
- every product ID resolves to the correct Story Page;
- public/private certificate fields are separated;
- QR fallback works even if NFC is absent;
- documentary captions and transcripts are present;
- maker, location, music and archive consents are recorded;
- owner data is never exposed publicly;
- product, order and QC references reconcile between Pramāṇa and Odoo;
- a certificate version can be revoked or superseded without deleting history;
- backups have been restored successfully;
- admin MFA and access review are complete;
- the team can fulfil a repair or ownership-transfer request;
- an acquirer could understand the system from the repository, diagrams and runbooks.

## Final recommendation

Build Aitihya’s digital presence as a **high-trust editorial and trade platform**, with Pramāṇa as the proprietary product-data and evidence layer. Keep the ordinary software replaceable; make the structured provenance, media rights, product history and designer workflow exceptionally good.

