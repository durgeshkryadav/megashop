# GitHub Copilot Instructions

## General Behavior

You are a code agent. Always make changes directly without asking for permission unless the user explicitly asks you not to make changes. Be proactive and implement solutions rather than just suggesting them.

## Code Comments

### Guiding Principles
1. **Minimize comments:** code should be self-explanatory; comment only when needed.
2. **Clarity through naming:** use clear, descriptive names for variables, functions, classes, components.
3. **Comment the "why," not the "what":** explain *why* an approach was taken, especially for non-obvious logic or trade-offs. Don't restate what the code does.
4. **Avoid obvious comments:** don't comment on simple or standard code; assume the reader knows language basics.
5. **Consistency:** keep comment and doc style consistent across the codebase.

### When to Add Comments
- **Complex logic:** for algorithms or flows not clear from code.
- **Non-obvious decisions (the "why"):** explain reasoning behind design choices, workarounds, or optimizations, especially if nonstandard or counterintuitive.
- **Business rules/context:** briefly note domain-specific rules or context driving code.
- **Security/performance notes:** note important security or performance implications if not obvious.
- **TODOs/FIXMEs:** use `// todo:` or `// fixme:` for future work, with brief context. Use `// @ts-expect-error todo: ...` only if you tried and failed to fix a TypeScript error, and disabling it is safe.
- **JSDoc (public APIs):**
  - Use doc comments for public functions or modules to describe purpose and usage.
  - Explain conceptual behavior, non-obvious details, or the "why" if needed.
  - **Skip redundant tags** like `@param`/`@return` if TypeScript types are clear.
  - Use tags only when types aren't enough (e.g., constraints, units, expected format).
  - Add `@example` for complex APIs.

### When NOT to Add Comments
- **Restating code:** e.g., `// increment i` above `i++`.
- **Explaining syntax:** e.g., `// loop through array` above `for (const item of items) { ... }`.
- **Obvious names:** e.g., `// user's name` above `const username = ...`.
- **Commented-out code:** delete dead code; git tracks history.
- **End-of-line noise:** avoid trivial comments on code lines.
- **Outdated comments:** remove or update comments that no longer match the code.

### Comment Style
- **All comments must be written in lowercase, even at the beginning of sentences.** The only exception is for code identifiers (like function or variable names), which should keep their original casing.
- **Full informal freedom:** use informal, conversational, or humorous language in comments and docs—just like a real, quirky senior dev. Puns, memes, pop culture, playful banter are all fair game, as long as it's not offensive or unprofessional.
- **Be yourself:** comments can have personality. If you want to drop a "don't touch this unless you like pain", that's totally fine.
- **Clarity first:** even with informality, make sure the comment's intent is clear and helpful to both beginners and seniors.

**Important:** These rules apply only to comments (e.g., `//`, `/** */`, `{/* */}`). Do not apply these rules to end-user content like frontend text.

## Working with Drizzle ORM

### Debugging: `TypeError: Cannot read properties of undefined (reading 'referencedTable')`

This error happens when Drizzle can't find relationship metadata for a relational query (e.g., `db.query.someTable.findMany({ with: { relatedTable: true } })`).

**Checklist:**

1. **Missing `relations` definition:**
   - Define a `relations` object for the source table (e.g., `userRelations = relations(userTable, ({ many }) => ({ uploads: many(uploadsTable) }))`).
   - `.references()` only sets up the DB foreign key; it doesn't create Drizzle's relation for queries.

2. **Relations not exported:**
   - Export all `relations` objects from schema files.
   - Ensure your schema barrel file (e.g., `src/db/schema/index.ts`) exports all tables and relations.

3. **Wrong schema passed to Drizzle:**
   - When initializing, pass a schema object with both tables and relations:
     ```typescript
     import * as schema from './schema';
     const db = drizzle(conn, { schema });
     ```

4. **Build issues:**
   - In monorepos or complex setups, make sure all schema/relation files are included in the build.

5. **Multiple Drizzle instances:**
   - Use a single shared Drizzle client (e.g., export `db` from `src/db/index.ts`).

## Working with UploadThing

### Key Files
- **Config:** `core.ts` (upload routes, middleware, db)
- **API handler:** `route.ts` (webhook/callback)
- **DB schema:** `tables.ts` (`uploads` table, `type` enum)
- **DB types:** `types.ts` (`MediaUpload`)
- **Frontend UI:** `page.client.tsx` (`UploadButton`)
- **Media API:** `/api/media/route.ts` (fetch media)
- **Display:** `bento-media-gallery.tsx` (renders images/videos)
- **Auth:** `~/lib/auth`

### Core Concepts
- **Routes:** defined in `core.ts` with `createUploadthing()`. Each route (e.g., `imageUploader`, `videoUploader`) sets allowed types, size, count.
- **Auth:** middleware in `core.ts` uses `auth.api.getSession` to check user; throws if unauthorized.
- **DB insert:** `onUploadComplete` inserts into `uploads` table, sets `type` based on route, uses `createId()` for `id`.
- **Frontend:** `UploadButton` for each endpoint; `onClientUploadComplete` refetches media via `/api/media`.
- **Media fetch:** `/api/media/route.ts` authenticates, fetches uploads for user, returns JSON.
- **Display:** `bento-media-gallery.tsx` renders `<img>` or `<video>` based on `type`.

### Tips
- Check `core.ts` for routes, middleware, and upload logic.
- Set `type` in `onUploadComplete` based on route.
- **Adding new media types:** update `core.ts`, `tables.ts` enum, run migrations, add `UploadButton`, update display and API as needed.
- Media fetch requires auth.
- **Error handling:** `onUploadComplete` logs/throws; production may need more.

## Project Structure

This is a Next.js project with the following key directories:
- **`src/app/`** - Next.js app router pages and layouts
- **`src/api/`** - API services
- **`src/db/`** - Database schema and queries (Drizzle ORM)
- **`src/lib/`** - Utilities, hooks, and helpers
- **`src/ui/`** - UI components (primitives and composed components)
- **`public/`** - Static assets

### Authentication
- Auth logic is in `src/lib/auth.ts`, `auth-client.ts`, `auth-db.ts`
- Auth types in `auth-types.ts`

### Database
- Schema definitions in `src/db/schema/`
- Use Drizzle ORM for all database operations
- Export all tables and relations from schema barrel file

### UI Components
- Primitives in `src/ui/primitives/`
- Composed components in `src/ui/components/`
- Use consistent naming and structure

By following these instructions, maintain a clean, readable, and maintainable codebase with personality and clarity.
