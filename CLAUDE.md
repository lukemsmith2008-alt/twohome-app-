# TwoHome Services field app: project notes

A family-run home watch business serving homes in Central Florida and the Indianapolis area.
This repo is the team's field app, served by GitHub Pages from the `main` branch root.
The owner is not a developer, so explain changes in plain language.

## How the app is built
- `index.html` holds the whole app: plain HTML/CSS/vanilla JS with no build step. jsPDF 2.5.1 is embedded inline.
- `supabase.js` is the vendored Supabase JS v2 UMD build, loaded same-origin so it works offline.
- `sw.js` is the service worker: network-first with a 4 s timeout, falling back to the cache. Bump `CACHE` when the cached file list changes.
- Local storage is IndexedDB `twohome-pilot`, with stores `kv` (`data` = all records, `sync` = sync state) and `photos` (blobs).
- Cloud is Supabase project `hbgsyaofudiuajobwfaa` (East US). The publishable key is in `index.html` and is public by design; row-level security protects the data.
  - Table `team(email, name, role owner|member)` with helper functions `is_team()` and `is_owner()`.
  - Table `records(id, kind, data jsonb, deleted, updated_at, updated_by)` is a document store. Kinds: home, visit, charge, statement, settings. A trigger sets `updated_at`.
  - Private storage bucket `photos`, with objects named `<photoId>.jpg`. Team-only policies.
- Sync works like this: local-first, push changed records by hash, then pull rows with `updated_at >= lastPull`. Last write wins. Deletes are soft (`deleted=true`).
- Settings fields `pinHash` and `me` stay on the device and are never synced.

## Features
Visits with a per-home checklist (Indy winter and FL summer templates); statuses OK / Serviced / Soon / Urgent; required time-stamped photo proof (arrival, departure, before/after, flagged items); PDF visit reports and PDF statements shared through the Web Share API; Billing tab (plan rate, add-on charges, statements, payment links to Square or Stripe; the app never stores card data); home profile with baseline photos; PIN lock; offline mode; team sign-in with email and password (email confirmation is turned off in Supabase).

## Roadmap
- Phase 2: scheduling, generating visits from each plan's frequency and assigning them to team members.
- Phase 3: client portal (clients sign in, view reports, approve recommendations).
- Phase 4: built-in payments (Stripe Checkout or subscriptions, plus a webhook that marks invoices paid).
- Before passing about 25 clients: have a developer do a security review.

## Conventions
- Keep everything in `index.html` unless there's a strong reason not to. The owner deploys by pushing to `main`.
- Test with Playwright plus a mock Supabase (route `supabase.js` to a mock) before pushing, because the dev sandbox may not reach supabase.co.
- Never commit secrets (Supabase service_role key, database password) or client data.
