# افزی

افزی لینک را کوتاه می‌کند. یک نشانی بلند وارد می‌شود و یک لینک کوتاه برمی‌گردد. باز کردن لینک کوتاه به همان صفحه می‌رود. صفحهٔ مقصد ساخته نمی‌شود؛ مقصد همان نشانی ذخیره‌شده است.

سایت: [afzi.ir](https://afzi.ir)

وب و API افزی داخل برنامهٔ وب کسب‌افزار است، فقط روی مسیرهای افزی. بقیهٔ پیشخوان کسب‌افزار موضوع این صفحه نیست.

## رفتار عمومی

صفحهٔ [afzi.ir](https://afzi.ir) می‌گوید نشانی بلند داده می‌شود و آدرس کوتاه گرفته می‌شود، و صفحه همان سایت مقصد است. صفحهٔ [امکانات](https://afzi.ir/features) این موارد را به‌عنوان رفتار فعلی فهرست می‌کند:

- لینک کوتاه، با نام دلخواه یا با کد ساخته‌شده
- QR برای هر لینک کوتاه، و QR ایستا برای وای‌فای، پیامک، ایمیل، واتساپ، تلگرام، و متن
- QR پویا: با عوض شدن مقصد، تصویر QR همان می‌ماند
- آمار کلیک: تعداد، کشور اگر سرور بگوید، دستگاه، سیستم‌عامل، مرورگر، زبان، سایت ارجاع، و بازهٔ زمانی
- موضوع و رنگ برای مرتب کردن لینک‌ها
- رمز قبل از رفتن به مقصد
- تاریخ انقضا، سقف کلیک، و نشانی جایگزین بعد از انقضا
- سازندهٔ UTM روی نشانی بلند
- هدف‌گیری کشور از کد کشور بازدیدکننده
- هدف‌گیری سکو: اندروید، iOS، ویندوز، مک، لینوکس
- چرخش چند مقصد با وزن
- کوتاه‌کردن جمعی تا ۲۰ نشانی
- `POST /api/afzi/links`
- بوکمارکلت

همان صفحه می‌گوید دامنهٔ اختصاصی، تیم، همکاری در فروش، پیکسل تبلیغاتی، و لینک‌در‌بیو جزو این محصول نیستند.

## English

Afzi shortens a link. A long address goes in. A short link comes out. Opening the short link sends the visitor to that stored address. Afzi does not build a new page for the destination.

Site: [afzi.ir](https://afzi.ir)

The Afzi web app and HTTP API live in the Kasbafzar web application, on the Afzi paths only.

The public features page at [afzi.ir/features](https://afzi.ir/features) lists current behavior: a short link with an optional custom name, a QR image per short link, a static QR for Wi-Fi, SMS, email, WhatsApp, Telegram, or plain text, a dynamic QR whose image stays put when the destination changes, click stats (count, country when the server provides it, device, operating system, browser, language, referrer, and a time range), topic and color, a password before redirect, expiry, a click cap, a fallback address after expiry, a UTM builder on the long address, country targeting, platform targeting (Android, iOS, Windows, Mac, Linux), weighted rotation across destinations, bulk shorten of up to 20 addresses, `POST /api/afzi/links`, and a bookmarklet. That page states that a custom domain, teams, affiliate, an ad pixel, and a link-in-bio page are not part of this product.

## Where the code lives

Routes and libraries below are files under the Kasbafzar web app.

- Redirect: `apps/web/src/app/afzi/[id]/route.ts` and `apps/web/src/app/afzi/[id]/[invoiceRef]/route.ts`
- Create and list: `apps/web/src/app/api/afzi/links/route.ts`
- Bulk: `apps/web/src/app/api/afzi/links/bulk/route.ts`
- Stats: `apps/web/src/app/api/afzi/links/stats/route.ts`
- QR for a code: `apps/web/src/app/api/afzi/qr/[code]/route.ts`
- Static QR: `apps/web/src/app/api/afzi/qr/encode/route.ts`
- Unlock: `apps/web/src/app/api/afzi/unlock/route.ts`
- Rules: `apps/web/src/lib/afzi/short-link.ts`
- Database access: `apps/web/src/lib/afzi/short-link-db.ts`
- Public pages: `apps/web/src/app/(afzi-public)/afzi-site/page.tsx`, `features/page.tsx`, `bookmarklet/page.tsx`, `contact/page.tsx`, `unlock/page.tsx`

The web package that serves these routes is `apps/web/package.json`: Next.js `^14.2.21`, React `^18.3.1`, TypeScript `^5.7.2`. Handlers set `runtime` to `nodejs`.

## Create rules

`apps/web/src/lib/afzi/short-link.ts` is the shared rule module. A destination must be `http` or `https`, trimmed, at most 2048 characters, with no whitespace, no username, and no password in the URL. `parseShortLinkDestination` returns null otherwise.

A code matches `^[a-zA-Z0-9_-]{4,64}$`. Reserved segments and a blocked-alias set are rejected. The blocked set in that file includes `about`, `admin`, `afzi`, `afzi-site`, `api`, `app`, `contact`, `stats`, `bulk`, `bookmarklet`, `unlock`, `features`, `faq`, `invoice`, `login`, `market`, `offer`, `privacy`, `store`, `terms`, and the other literals in `BLOCKED_ALIASES`.

A label is optional, at most 120 characters. Empty becomes null. An expiry must parse as a future time and must be within five years (`MAX_EXPIRY_MS` is five years in milliseconds). Kinds are `official` and `public`. `resolveCreateKind` gives an anonymous actor a `public` link.

`POST` and `GET` are exported from `apps/web/src/app/api/afzi/links/route.ts`. Persistence functions in `short-link-db.ts` include `createAfziShortLink`, `findAfziShortLink`, `resolveActiveRedirectUrl`, `recordAfziShortLinkClick`, `listOwnedShortLinks`, and `getOwnedShortLinkStats`.

## Redirect

`GET` on `apps/web/src/app/afzi/[id]/route.ts` loads the row with `findAfziShortLink`.

If the row status is `expired` and `expiredRedirect` parses as an http(s) URL, the response is `302` to that URL with `Cache-Control: no-store`.

If the row is `active`, `resolveAfziFirstSegment` treats a stored destination as a short link. When `passwordHash` is set, the handler reads the unlock cookie from `unlockCookieName(code)` and checks `unlockTokenMatches`. A mismatch redirects `302` to `/unlock?code=...` on the Afzi base URL, again `no-store`. A match calls `resolveActiveRedirectUrl`, records a click with `recordAfziShortLinkClick` and `clickMetaFromRequest`, and returns `302` to the target.

If there is no short link and no other resolved entity, an expired row without a fallback becomes HTTP `410` with the plain-text body `این لینک منقضی شده است`.

## Data

Prisma datasource provider is `postgresql` in `packages/db/schema.prisma`. Prisma Client is `6.19.3` in `packages/db/package.json`. `short-link-db.ts` imports `prisma` from `@kasbafzar/db`.

Model `AfziShortLink` fields include `code`, `destination`, `kind`, `label`, `topic`, `topicColor`, `passwordHash`, `expiresAt`, `maxClicks`, `expiredRedirect`, `destinations`, `geoRules`, `platformRules`, `clicks`, and `createdAt`. Model `AfziShortLinkClick` stores `country`, `device`, `os`, `browser`, `language`, `referrer`, and `createdAt` on a link. Those click columns are the stats the features page describes.

## QR

`GET /api/afzi/qr/encode?text=` in `apps/web/src/app/api/afzi/qr/encode/route.ts` rejects an empty string, a string longer than 1200 characters, and a `javascript:` prefix. It calls `generateQRBuffer` from `@kasbafzar/qr` and returns `image/png` with `Cache-Control: public, max-age=120`. The comment on that route says this image is a static QR, not a short link.

`packages/qr/package.json` depends on `qrcode` `^1.5.4`. `generateQRBuffer` in `packages/qr/src/index.ts` calls `QRCode.toBuffer` with width 200 and margin 1. `generateQRFile` writes a PNG at width 300 and margin 2.

## Boundaries

Hosting, credentials, and Kasbafzar modules other than these Afzi paths are omitted.

## پروژه‌های مرتبط

- [hamejoo-intro](https://github.com/mnhashemabadi/hamejoo-intro): همه‌جو نتیجهٔ جستجوی آگهی را یک‌جا نشان می‌دهد و جزئیات آگهی روی منبع اصلی می‌ماند.
- [alwer-intro](https://github.com/mnhashemabadi/alwer-intro): آلور بازار نیازمندی است؛ درخواست خرید ثبت می‌شود، فروشنده پیشنهاد قیمت می‌فرستد، و آگهی فروش هم در همان بازار است.
- [kasbafzar-intro](https://github.com/mnhashemabadi/kasbafzar-intro): کسب‌افزار پیشخوان فروش و گزارش مالی است؛ فروش، مشتری و هزینه ثبت می‌شود و فاکتور می‌تواند لینک پرداخت داشته باشد.
- [azadchi-intro](https://github.com/mnhashemabadi/azadchi-intro): آزادچی بازار نیازمندی مناطق آزاد و ویژهٔ اقتصادی است؛ آگهی در همان محدوده جستجو می‌شود و گفتگو داخل همان محصول است.
- [alweryar-intro](https://github.com/mnhashemabadi/alweryar-intro): آلوریار شبکهٔ همکاران آلور برای بررسی آگهی و همکاری در فروش است.
- [alwerchi-intro](https://github.com/mnhashemabadi/alwerchi-intro): آلورچی خانهٔ فروشگاه‌هایی است که کالا و موجودی‌شان در بازار آلور دیده می‌شود و خریدار در آلور می‌ماند.

## Related

- [hamejoo-intro](https://github.com/mnhashemabadi/hamejoo-intro): Hamejoo shows listing search results in one place, and the listing detail stays on the original source.
- [alwer-intro](https://github.com/mnhashemabadi/alwer-intro): Alwer is a classifieds marketplace: a buyer posts a request, sellers send price offers, and a sale listing can be posted on the same market.
- [kasbafzar-intro](https://github.com/mnhashemabadi/kasbafzar-intro): Kasbafzar is a sales desk and a financial report: sales, customers, and expenses are recorded, and an invoice can carry a payment link.
- [azadchi-intro](https://github.com/mnhashemabadi/azadchi-intro): Azadchi is a classifieds market for free zones and special economic zones: listings are searched in that area, and the conversation stays in the product.
- [alweryar-intro](https://github.com/mnhashemabadi/alweryar-intro): Alweryar is Alwer's collaborator network for listing review and for sales collaboration.
- [alwerchi-intro](https://github.com/mnhashemabadi/alwerchi-intro): Alwerchi is the home of shops whose goods and stock appear on the Alwer marketplace while the buyer stays on Alwer.

یادداشت مهندسی کوتاه‌تر: [docs/engineering.md](docs/engineering.md)

Shorter engineering note: [docs/engineering.md](docs/engineering.md)

