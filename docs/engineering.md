# Engineering note

## Overview

Afzi shortens a link. A long address goes in. A short link comes out. Opening the short link sends the visitor to that same page. The address can be a listing, a shop page, an invoice, or any other link.

## Stack

- Next.js ^14.2.21
- React ^18.3.1
- TypeScript ^5.7.2
- PostgreSQL
- Prisma Client 6.19.3
- qrcode ^1.5.4

The HTTP API and the redirects are Next.js route handlers on the Node.js runtime.

## Specialties

- Link shortening and redirect, with country, platform, and weighted-rotation targeting
- Dynamic QR images built from the short address, so the image survives a destination change
- Click stats by country, device, operating system, browser, language, and referrer domain
