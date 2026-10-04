# Engineering note

## Overview

Afzi shortens a link. A long address goes in. A short link comes out. Opening the short link sends the visitor to that same page. The address can be a listing, a shop page, an invoice, or any other link.

Afzi's web app and HTTP API live in the Kasbafzar web application, on the Afzi paths only. This note does not describe the rest of that application.

## Stack

From `apps/web/package.json`, which serves the Afzi routes:

- Next.js ^14.2.21
- React ^18.3.1
- TypeScript ^5.7.2

HTTP API routes are Next.js route handlers under `apps/web/src/app/api/afzi`. Redirect routes are under `apps/web/src/app/afzi`.

Database:

- PostgreSQL, Prisma datasource provider `postgresql` in `packages/db/schema.prisma`
- Prisma Client 6.19.3 in `packages/db/package.json`
- Short links are the Prisma model `AfziShortLink`. Access is in `apps/web/src/lib/afzi/short-link-db.ts`, which imports `prisma` from `@kasbafzar/db`

QR images:

- Library `qrcode` ^1.5.4 in `packages/qr/package.json`
- Afzi encode route `apps/web/src/app/api/afzi/qr/encode/route.ts` imports `generateQRBuffer` from `@kasbafzar/qr`

## Specialties

- Link shortening and redirect: Next.js route handlers and the Prisma model `AfziShortLink`
- QR images: `qrcode`, called from the Afzi encode route

## Boundaries

This note covers the short-link web app, its HTTP API, and the short-link table. It does not describe hosting, credentials, or other Kasbafzar modules.
