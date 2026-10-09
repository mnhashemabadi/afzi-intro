# افزی

افزی را ساختم تا یک نشانی بلند به لینک کوتاه تبدیل شود و باز کردن آن لینک همان صفحه را باز کند. صفحهٔ جدا برای مقصد نمی‌سازم. مقصد همان نشانی ذخیره‌شده است. اگر صفحه‌ای در میانه می‌ساختم، افزی یک سایت دیگر می‌شد، نه کوتاه‌کننده.

سایت: [afzi.ir](https://afzi.ir)

## آنچه صفحهٔ امکانات می‌گوید الان کار می‌کند

روی [afzi.ir/features](https://afzi.ir/features) نوشتم این‌ها رفتار فعلی است، نه فهرست آرزو:

- لینک کوتاه با نام دلخواه، یا با کدی که خودم می‌سازم. سقف تعداد نگذاشتم.
- QR برای هر لینک کوتاه، و QR ایستا برای وای‌فای، پیامک، ایمیل، واتساپ، تلگرام و متن.
- QR پویا: با عوض شدن مقصد، تصویر QR همان می‌ماند.
- آمار کلیک: تعداد، کشور، دستگاه، سیستم‌عامل، مرورگر، زبان، سایت ارجاع، و بازهٔ زمانی.
- موضوع و رنگ برای مرتب کردن لینک‌ها. مثل پوشه است، نه تیم و نه صورتحساب.
- قفل لینک: بازدیدکننده پیش از رفتن به مقصد عبارت عبور را وارد می‌کند.
- تاریخ انقضا، سقف کلیک، و نشانی جایگزین بعد از انقضا.
- سازندهٔ UTM روی نشانی بلند.
- هدف‌گیری کشور از کد کشور بازدیدکننده.
- هدف‌گیری سکو: اندروید، iOS، ویندوز، مک، لینوکس.
- چرخش چند مقصد با وزن.
- کوتاه‌کردن جمعی تا ۲۰ نشانی.
- API عمومی برای ساخت لینک.
- بوکمارکلت.

## پشتهٔ فنی

وب و API افزی با Next.js ^14.2.21، React ^18.3.1 و TypeScript ^5.7.2 نوشته شده‌اند. هندلرهای مسیر روی runtime نود اجرا می‌شوند، نه edge. داده روی PostgreSQL است و از راه Prisma Client 6.19.3 خوانده و نوشته می‌شود. تصویر QR با `qrcode` ^1.5.4 ساخته می‌شود.

## چه نشانی‌ای ذخیره می‌شود

مقصد فقط `http` یا `https` پذیرفته می‌شود: بعد از حذف فاصلهٔ دو سر، حداکثر ۲۰۴۸ نویسه، بدون فاصله در میانه، و بدون اطلاعات ورود داخل خود URL. نشانی‌ای را که نام کاربری و اطلاعات ورود را در خودش دارد ذخیره نمی‌کنم، چون آن رشته خودش محرمانه است.

کد کوتاه ۴ تا ۶۴ نویسه از حرف لاتین، رقم، خط تیره و زیرخط است. واژه‌های رزرو رد می‌شوند تا کد کوتاه جای مسیرهای خود سایت را نگیرد. برچسب اختیاری است و حداکثر ۱۲۰ نویسه دارد. تاریخ انقضا باید در آینده باشد و از پنج سال جلوتر نرود. لینک عمومی http(s) نه به فروشگاه نیاز دارد و نه به نشست.

## هدایت

اگر لینک منقضی باشد و نشانی جایگزین داشته باشد، پاسخ `302` به همان نشانی است با `Cache-Control: no-store`. اگر صفحهٔ منقضی کش شود، نشانی جایگزین همان‌جا ثابت می‌ماند. لینک منقضی بدون نشانی جایگزین پاسخ `410` می‌دهد با متن «این لینک منقضی شده است».

اگر لینک قفل باشد، بازدیدکننده به صفحهٔ باز کردن قفل می‌رود، باز هم با `no-store`، و بعد از وارد کردن عبارت درست به مقصد می‌رسد.

مقصد به این ترتیب انتخاب می‌شود: مقصد اصلی، بعد قاعدهٔ کشور، بعد سکو از روی User-Agent، بعد چرخش وزنی. کلیک ثبت می‌شود و پاسخ `302` به مقصد است. کشور را از پایگاه جغرافیایی شخص ثالث نمی‌گیرم؛ کد دوحرفی کشور را از لبهٔ شبکه می‌خوانم و اگر نبود، قاعدهٔ کشور اعمال نمی‌شود. دستگاه و سیستم‌عامل و مرورگر را از User-Agent برچسب می‌زنم و زبان را از `Accept-Language`. از ارجاع فقط دامنه را نگه می‌دارم، حداکثر ۱۲۰ نویسه، نه کل نشانی صفحهٔ ارجاع را.

## QR چرا با عوض شدن مقصد ثابت می‌ماند

QR لینک کوتاه نشانی دلخواه نمی‌پذیرد. فقط لینک فعال را پیدا می‌کند و تصویر PNG را از خود نشانی کوتاه می‌سازد، نه از مقصد. برای همین عوض کردن مقصد تصویر را عوض نمی‌کند. لینک منقضی اینجا `410` برمی‌گرداند و کد نامعتبر `404`. کش این PNG را `public, max-age=300` گذاشتم. QR ایستا برای وای‌فای و پیامک و بقیه مسیر جدای خودش را دارد.

کلیک‌ها در جدول جدا ذخیره می‌شوند، با ایندکس روی لینک و زمان، تا آمار بازه‌ای بدون پیمایش کامل جدول بیاید.

## English

I built Afzi so a long address becomes a short link, and opening that link opens the same page. I do not build a destination page. The destination is the stored address. A landing page in the middle would make Afzi another site, not a shortener.

Site: [afzi.ir](https://afzi.ir)

### What the features page says works now

On [afzi.ir/features](https://afzi.ir/features) I wrote that these are the current behavior, not a wish list:

- A short link with a name you choose, or with a code I generate. I did not put a count cap.
- A QR for each short link, and a static QR for Wi-Fi, SMS, email, WhatsApp, Telegram, or plain text.
- A dynamic QR: when the destination changes, the QR image stays.
- Click stats: count, country, device, operating system, browser, language, referrer, and a time range.
- A topic and a color for sorting links. That is a folder, not a team and not an invoice.
- A link lock: the visitor enters a passphrase before the redirect.
- An expiry date, a click cap, and a fallback address after expiry.
- A UTM builder on the long address.
- Country targeting from the visitor's country code.
- Platform targeting: Android, iOS, Windows, Mac, Linux.
- Weighted rotation across destinations.
- Bulk shorten of up to 20 addresses.
- A public API for creating links.
- A bookmarklet.

### Stack

The Afzi web app and API are written with Next.js ^14.2.21, React ^18.3.1, and TypeScript ^5.7.2. Route handlers run on the Node.js runtime, not edge. Data is on PostgreSQL, read and written through Prisma Client 6.19.3. QR images are built with `qrcode` ^1.5.4.

### What address gets stored

Only an `http` or `https` destination is accepted: trimmed, at most 2048 characters, no whitespace inside, and no credentials embedded in the URL. I do not store a URL that carries a username and credentials, because that string is itself confidential.

A short code is 4 to 64 characters of Latin letters, digits, hyphen, and underscore. Reserved words are rejected so a short code cannot take a path of the site itself. A label is optional and at most 120 characters. An expiry must be in the future and inside five years. A public http(s) link does not need a store or a session.

### Redirect

If a link is expired and has a fallback address, the response is `302` to that address with `Cache-Control: no-store`. A cached expired page would freeze the fallback. An expired link without a fallback returns `410` with the plain text «این لینک منقضی شده است».

If a link is locked, the visitor goes to the unlock page, again `no-store`, and reaches the destination after entering the right phrase.

The target is picked in this order: the main destination, then the country rule, then the platform from the User-Agent, then weighted rotation. The click is recorded and the response is `302` to the target. I do not call a third-party geo database; I read the two-letter country code from the network edge, and when it is missing the country rule does not apply. I label device, operating system, and browser from the User-Agent, and language from `Accept-Language`. From the referrer I keep only the domain, at most 120 characters, not the full referring URL.

### Why the QR stays when the destination changes

The short-link QR does not accept an arbitrary URL. It finds the active link and builds the PNG from the short address itself, not from the destination. That is why changing the destination does not change the image. An expired link here returns `410`, and an invalid code returns `404`. I cache that PNG as `public, max-age=300`. The static QR for Wi-Fi, SMS, and the rest has its own path.

Clicks are stored in their own table with an index on link and time, so a time-range stat does not scan the whole table.

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
