# افزی

افزی را ساختم تا یک نشانی بلند به لینک کوتاه تبدیل شود و باز کردن آن لینک همان صفحه را باز کند. صفحهٔ جدا برای مقصد نمی‌سازم. مقصد همان نشانی ذخیره‌شده است. اگر صفحه‌ای در میانه می‌ساختم، افزی یک سایت دیگر می‌شد، نه کوتاه‌کننده.

سایت: [afzi.ir](https://afzi.ir)

وب و API افزی داخل اپ وب کسب‌افزار است و فقط روی مسیرهای افزی. بقیهٔ پیشخوان کسب‌افزار موضوع این صفحه نیست.

## آنچه صفحهٔ امکانات می‌گوید الان کار می‌کند

روی [afzi.ir/features](https://afzi.ir/features) نوشتم این‌ها رفتار فعلی است، نه فهرست آرزو:

- لینک کوتاه با نام دلخواه، یا با کدی که خودم می‌سازم. سقف تعداد نگذاشتم.
- QR برای هر لینک کوتاه، و QR ایستا برای وای‌فای، پیامک، ایمیل، واتساپ، تلگرام و متن.
- QR پویا: با عوض شدن مقصد، تصویر QR همان می‌ماند.
- آمار کلیک: تعداد، کشور اگر سرور آن را بدهد، دستگاه، سیستم‌عامل، مرورگر، زبان، سایت ارجاع، و بازهٔ زمانی.
- موضوع و رنگ برای مرتب کردن لینک‌ها. مثل پوشه است، نه تیم و نه صورتحساب.
- رمز قبل از رفتن به مقصد.
- تاریخ انقضا، سقف کلیک، و نشانی جایگزین بعد از انقضا.
- سازندهٔ UTM روی نشانی بلند.
- هدف‌گیری کشور از کد کشور بازدیدکننده.
- هدف‌گیری سکو: اندروید، iOS، ویندوز، مک، لینوکس.
- چرخش چند مقصد با وزن.
- کوتاه‌کردن جمعی تا ۲۰ نشانی.
- `POST /api/afzi/links`.
- بوکمارکلت.

همان صفحه می‌گوید دامنهٔ اختصاصی، تیم، همکاری در فروش، پیکسل تبلیغاتی و لینک در بیو جزو این محصول نیستند، چون زیرساختش را اینجا نیاوردم.

## کد کجاست

همهٔ این‌ها فایل‌های اپ وب کسب‌افزار است.

- هدایت: `apps/web/src/app/afzi/[id]/route.ts` و `apps/web/src/app/afzi/[id]/[invoiceRef]/route.ts`
- ساخت و فهرست: `apps/web/src/app/api/afzi/links/route.ts`
- جمعی: `apps/web/src/app/api/afzi/links/bulk/route.ts` با `MAX_BULK` برابر ۲۰
- آمار: `apps/web/src/app/api/afzi/links/stats/route.ts`
- QR یک کد: `apps/web/src/app/api/afzi/qr/[code]/route.ts`
- QR ایستا: `apps/web/src/app/api/afzi/qr/encode/route.ts`
- باز کردن رمز: `apps/web/src/app/api/afzi/unlock/route.ts`
- قاعده: `apps/web/src/lib/afzi/short-link.ts`
- پایگاه: `apps/web/src/lib/afzi/short-link-db.ts`
- صفحه‌های عمومی: `apps/web/src/app/(afzi-public)/afzi-site/`

بستهٔ وبی که این مسیرها را در اختیار می‌گذارد `apps/web/package.json` است: Next.js ^14.2.21، React ^18.3.1، TypeScript ^5.7.2. هندلرها `runtime` را `nodejs` گذاشته‌اند. Prisma در `packages/db/schema.prisma` با provider برابر `postgresql` است و Prisma Client در `packages/db/package.json` نسخهٔ `6.19.3` است. `short-link-db.ts` از `@kasbafzar/db` وارد می‌کند.

## چه نشانی‌ای ذخیره می‌شود

`parseShortLinkDestination` فقط `http` یا `https` را برمی‌گرداند: بعد از trim، حداکثر ۲۰۴۸ نویسه، بدون فاصله، بدون نام کاربری و بدون رمز داخل URL، و با hostname. در غیر این صورت null است. لینک با یوزر و پسورد داخل URL را ذخیره نمی‌کنم چون آن رشته خودش راز است.

کد باید با `^[a-zA-Z0-9_-]{4,64}$` مطابق باشد. قطعه‌های رزرو و مجموعهٔ `BLOCKED_ALIASES` رد می‌شوند تا کد کوتاه جای مسیر خود سایت را نگیرد؛ از جمله `api`، `app`، `invoice`، `login`، `features`، `unlock` و بقیهٔ لفظ‌های همان مجموعه. برچسب اختیاری است و حداکثر ۱۲۰ نویسه دارد. اگر خالی باشد null می‌شود. تاریخ انقضا باید در آینده باشد و از پنج سال جلوتر نرود (`MAX_EXPIRY_MS`). نوع‌ها `official` و `public` هستند. `resolveCreateKind` به درخواست‌کنندهٔ ناشناس فقط لینک `public` می‌دهد. لینک `official` را فقط درخواست‌کنندهٔ یکپارچه‌سازی می‌تواند بخواهد. یک لینک عمومی http(s) نه به فروشگاه نیاز دارد و نه به نشست.

رمز عبور این مرحله را با `scrypt` در `short-link-password.ts` هش می‌کنم و متن رمز را ذخیره نمی‌کنم. مدل `AfziShortLink` فیلد `passwordHash` را همین‌طور توضیح می‌دهد. `ownerKeyHash` هش کلید مالک است که یک بار هنگام ساخت برمی‌گردد، نه رمز ورود.

## هدایت

`GET` روی `apps/web/src/app/afzi/[id]/route.ts` ردیف را با `findAfziShortLink` می‌خواند.

اگر وضعیت `expired` باشد و `expiredRedirect` یک URL از نوع http(s) باشد، پاسخ `302` به همان نشانی است با `Cache-Control: no-store`. اگر صفحهٔ منقضی کش شود، نشانی جایگزین همان‌جا ثابت می‌ماند.

اگر ردیف `active` باشد و `passwordHash` داشته باشد، کوکی بازکردن را با `unlockCookieName(code)` می‌خوانم و `unlockTokenMatches` را چک می‌کنم. ناسازگاری، `302` به `/unlock?code=...` روی مبدأ افزی است، باز هم `no-store`. `POST /api/afzi/unlock` رمز را با `verifyLinkPassword` می‌سنجد و در صورت درستی کوکی را می‌گذارد.

اگر رمز مانع نباشد، `resolveActiveRedirectUrl` مقصد را این‌طور انتخاب می‌کند: اول `destination`، بعد قاعدهٔ کشور (`pickGeoUrl`)، بعد سکو از User-Agent (`pickPlatformUrl`)، بعد چرخش وزنی (`pickRotationUrl`). سپس `recordAfziShortLinkClick` و `302` به مقصد. کشور را از پایگاه جغرافیایی شخص ثالث نمی‌گیرم. `countryFromRequest` اولین هدر لبه‌ای را که کد دوحرفی باشد برمی‌دارد: `cf-ipcountry`، `x-vercel-ip-country`، `x-country-code`، `x-geo-country`. `XX` و `T1` را دور می‌ریزم. اگر هیچ‌کدام نبود کشور خالی می‌ماند و قاعدهٔ کشور اعمال نمی‌شود. دستگاه و سیستم‌عامل و مرورگر را از User-Agent برچسب می‌زنم. زبان از `accept-language` است. ارجاع را به hostname کوتاه می‌کنم، حداکثر ۱۲۰ نویسه، و کل URL ارجاع را نگه نمی‌دارم.

اگر لینک کوتاه و موجودیت دیگری در میان نباشد، ردیف منقضی بدون نشانی جایگزین پاسخ `410` می‌دهد با متن `این لینک منقضی شده است`. اگر کد به هیچ موجودیتی نخورد، پاسخ `302` به مبدأ کسب‌افزار است. مسیر `[id]/[invoiceRef]` را جدا گذاشتم تا ارجاع فاکتور با کد لینک کوتاه قاطی نشود.

## QR چرا با عوض شدن مقصد ثابت می‌ماند

`GET /api/afzi/qr/[code]` URL دلخواه نمی‌پذیرد. فقط لینک فعال را پیدا می‌کند و PNG را از نشانی کوتاه می‌سازد: `getAfziBaseUrl()` به‌علاوهٔ کد، با `generateQRBuffer` از `@kasbafzar/qr` (`qrcode` ^1.5.4). برای همین عوض کردن `destination` تصویر را عوض نمی‌کند. لینک منقضی اینجا `410` برمی‌گرداند و کد نامعتبر `404`. کش این PNG را `public, max-age=300` گذاشتم. QR ایستا (وای‌فای و پیامک و بقیه) مسیر `encode` است، نه این مسیر.

## جدول

`AfziShortLink` کد یکتا، مقصد، نوع `official` یا `public`، برچسب، موضوع و رنگ موضوع، فروشگاه و کاربر اختیاری، `ownerKeyHash`، `passwordHash`، `expiresAt`، `maxClicks`، `expiredRedirect`، `destinations` برای چرخش، `geoRules` و `platformRules` را دارد. کلیک‌ها مدل `AfziShortLinkClick` هستند: کشور، دستگاه، سیستم‌عامل، مرورگر، زبان، ارجاع، زمان، با ایندکس روی لینک و زمان تا آمار بازه‌ای بدون پیمایش کامل جدول بیاید.

میزبانی و رمزها را اینجا نیاوردم.

## English

I built Afzi so a long address becomes a short link, and opening that link opens the same page. I do not build a destination page. The destination is the stored address. A landing page in the middle would make Afzi another site, not a shortener.

Site: [afzi.ir](https://afzi.ir)

The Afzi web app and HTTP API live inside the Kasbafzar web app, on the Afzi paths only. The rest of that desk is not the subject of this page.

### What the features page says works now

On [afzi.ir/features](https://afzi.ir/features) I wrote that these are the current behavior, not a wish list:

- A short link with a name you choose, or with a code I generate. I did not put a count cap.
- A QR for each short link, and a static QR for Wi-Fi, SMS, email, WhatsApp, Telegram, or plain text.
- A dynamic QR: when the destination changes, the QR image stays.
- Click stats: count, country when the server provides it, device, operating system, browser, language, referrer, and a time range.
- A topic and a color for sorting links. That is a folder, not a team and not an invoice.
- A password before the redirect.
- An expiry date, a click cap, and a fallback address after expiry.
- A UTM builder on the long address.
- Country targeting from the visitor's country code.
- Platform targeting: Android, iOS, Windows, Mac, Linux.
- Weighted rotation across destinations.
- Bulk shorten of up to 20 addresses.
- `POST /api/afzi/links`.
- A bookmarklet.

That page says a custom domain, teams, affiliate, an ad pixel, and a link-in-bio page are not part of this product, because I did not build that infrastructure here.

### Where the code is

All of the following are files in the Kasbafzar web app.

- Redirect: `apps/web/src/app/afzi/[id]/route.ts` and `apps/web/src/app/afzi/[id]/[invoiceRef]/route.ts`
- Create and list: `apps/web/src/app/api/afzi/links/route.ts`
- Bulk: `apps/web/src/app/api/afzi/links/bulk/route.ts` with `MAX_BULK` of 20
- Stats: `apps/web/src/app/api/afzi/links/stats/route.ts`
- QR for a code: `apps/web/src/app/api/afzi/qr/[code]/route.ts`
- Static QR: `apps/web/src/app/api/afzi/qr/encode/route.ts`
- Unlock: `apps/web/src/app/api/afzi/unlock/route.ts`
- Rules: `apps/web/src/lib/afzi/short-link.ts`
- Database: `apps/web/src/lib/afzi/short-link-db.ts`
- Public pages: `apps/web/src/app/(afzi-public)/afzi-site/`

The web package that serves these routes is `apps/web/package.json`: Next.js ^14.2.21, React ^18.3.1, TypeScript ^5.7.2. Handlers set `runtime` to `nodejs`. Prisma in `packages/db/schema.prisma` uses provider `postgresql`, and Prisma Client in `packages/db/package.json` is `6.19.3`. `short-link-db.ts` imports from `@kasbafzar/db`.

### What address gets stored

`parseShortLinkDestination` returns only `http` or `https`: trimmed, at most 2048 characters, no whitespace, no username and no password in the URL, and a hostname. Otherwise it returns null. I do not store a URL that already carries a username and password, because that string is itself a secret.

A code must match `^[a-zA-Z0-9_-]{4,64}$`. Reserved segments and the `BLOCKED_ALIASES` set are rejected so a short code cannot take a path of the site itself, including `api`, `app`, `invoice`, `login`, `features`, `unlock`, and the other literals in that set. A label is optional and at most 120 characters. Empty becomes null. An expiry must be in the future and inside five years (`MAX_EXPIRY_MS`). Kinds are `official` and `public`. `resolveCreateKind` gives an anonymous actor only a `public` link. An `official` link can be requested only by the integration actor. A public http(s) link does not need a store or a session.

I hash the gate password with `scrypt` in `short-link-password.ts` and I do not store the password text. The `AfziShortLink` model describes `passwordHash` that way. `ownerKeyHash` is a hash of the owner key returned once at creation, not a login password.

### Redirect

`GET` on `apps/web/src/app/afzi/[id]/route.ts` loads the row with `findAfziShortLink`.

If status is `expired` and `expiredRedirect` parses as an http(s) URL, the response is `302` to that URL with `Cache-Control: no-store`. A cached expired page would freeze the fallback.

If the row is `active` and `passwordHash` is set, I read the unlock cookie from `unlockCookieName(code)` and check `unlockTokenMatches`. A mismatch is `302` to `/unlock?code=...` on the Afzi origin, again `no-store`. `POST /api/afzi/unlock` checks the password with `verifyLinkPassword` and sets the cookie when it matches.

When the link is open, `resolveActiveRedirectUrl` picks the target in this order: start from `destination`, then the country rule (`pickGeoUrl`), then the platform from the User-Agent (`pickPlatformUrl`), then weighted rotation (`pickRotationUrl`). Then `recordAfziShortLinkClick` and `302` to the target. I do not call a third-party geo database. `countryFromRequest` takes the first edge header that is a two-letter code: `cf-ipcountry`, `x-vercel-ip-country`, `x-country-code`, `x-geo-country`. I drop `XX` and `T1`. If none is present, country stays empty and the country rule does not apply. I label device, operating system, and browser from the User-Agent. Language comes from `accept-language`. I keep the referrer as a hostname, at most 120 characters, and I do not keep the full referrer URL.

If there is no short link and no other entity, an expired row without a fallback is HTTP `410` with the plain text `این لینک منقضی شده است`. If the code is not an entity at all, the response is `302` to the Kasbafzar origin. I kept `[id]/[invoiceRef]` separate so an invoice referral does not collide with a short-link code.

### Why the QR stays when the destination changes

`GET /api/afzi/qr/[code]` does not accept an arbitrary URL. It finds the active link and builds the PNG from the short address: `getAfziBaseUrl()` plus the code, through `generateQRBuffer` from `@kasbafzar/qr` (`qrcode` ^1.5.4). That is why changing `destination` does not change the image. An expired link here returns `410`, and an invalid code returns `404`. I cache that PNG as `public, max-age=300`. The static QR (Wi-Fi, SMS, and the rest) is the `encode` route, not this one.

### Table

`AfziShortLink` has a unique code, destination, kind `official` or `public`, label, topic and topic color, optional store and user, `ownerKeyHash`, `passwordHash`, `expiresAt`, `maxClicks`, `expiredRedirect`, `destinations` for rotation, `geoRules`, and `platformRules`. Clicks are `AfziShortLinkClick`: country, device, operating system, browser, language, referrer, and time, with an index on link and time so a time-range stat does not scan the whole table.

I am not putting hosting or credentials here.

## پروژه‌های مرتبط

- [hamejoo-intro](https://github.com/mnhashemabadi/hamejoo-intro): همه‌جو را برای بازدیدکننده‌ای ساختم که با یک عبارت آگهی را یک‌جا ببیند و برای جزئیات به سایت منبع برود.
- [alwer-intro](https://github.com/mnhashemabadi/alwer-intro): آلور را برای خریداری ساختم که درخواست بنویسد و پیشنهاد قیمت‌ها را کنار هم ببیند؛ آگهی فروش هم روی همان بازار است.
- [kasbafzar-intro](https://github.com/mnhashemabadi/kasbafzar-intro): کسب‌افزار را برای کسی ساختم که فروش و مشتری و هزینه را ثبت کند و فاکتور را با لینک پرداخت برای مشتری بفرستد.
- [azadchi-intro](https://github.com/mnhashemabadi/azadchi-intro): آزادچی را برای آگهی و جستجو در مناطق آزاد ساختم؛ گفتگو با طرف معامله داخل خود آزادچی می‌ماند.
- [alweryar-intro](https://github.com/mnhashemabadi/alweryar-intro): آلوریار را برای همکاری در بررسی آگهی و همکاری در فروش آلور ساختم.
- [alwerchi-intro](https://github.com/mnhashemabadi/alwerchi-intro): آلورچی را برای فروشگاهی ساختم که کالا و موجودی‌اش در بازار آلور دیده شود و خریدار در آلور بماند.

## Related

- [hamejoo-intro](https://github.com/mnhashemabadi/hamejoo-intro): I built Hamejoo for a visitor who types one phrase, sees listings in one place, and opens the original site for the listing itself.
- [alwer-intro](https://github.com/mnhashemabadi/alwer-intro): I built Alwer for a buyer who posts a request and compares price offers side by side; a sale listing sits on the same market.
- [kasbafzar-intro](https://github.com/mnhashemabadi/kasbafzar-intro): I built Kasbafzar for someone who records sales, customers, and expenses, and sends the customer an invoice with a payment link.
- [azadchi-intro](https://github.com/mnhashemabadi/azadchi-intro): I built Azadchi for listings and search inside free zones; the conversation with the other party stays in Azadchi.
- [alweryar-intro](https://github.com/mnhashemabadi/alweryar-intro): I built Alweryar for collaboration on listing review and on sales for Alwer.
- [alwerchi-intro](https://github.com/mnhashemabadi/alwerchi-intro): I built Alwerchi for a shop whose goods and stock show on the Alwer market while the buyer stays on Alwer.

یادداشت مهندسی کوتاه‌تر: [docs/engineering.md](docs/engineering.md)

Shorter engineering note: [docs/engineering.md](docs/engineering.md)
