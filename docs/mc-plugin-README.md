# BazzarMc — پلاگین سمت سرور ماینکرافت

پلاگین Spigot/Paper که فروشگاه ووکامرس (افزونهٔ **BazzarMc**) را به سرور ماینکرافت متصل می‌کند:
کد اتصال از پنل کاربری، اتصال با رمز یک‌بارمصرف ایمیلی از داخل بازی، تحویل خودکار آیتم‌ها/دستورها پس از پرداخت،
**منوی دریافت آیتم با کد خرید (GUI)**، **تعریف و همگام‌سازی رنک‌ها با سایت** و **یادآوری آیتم‌های تحویل‌گرفته‌نشده**.

* خروجی آماده: `../dist/BazzarMC-1.8.8.jar`
* سازگاری: Spigot / Paper / Purpur — نسخهٔ **1.16 تا 1.21**
* جاوا: بایت‌کد **Java 8** → روی سرورهای Java 8 تا Java 21 اجرا می‌شود
* وابستگی‌ها: `spigot-api` و `gson` با scope=provided (jar نهایی سبک و بدون shade)
* وابستگی اختیاری: **LuckPerms** — اگر نصب باشد، زمان انقضای واقعی گروه‌ها برای گزارش رنک خوانده می‌شود (بدون آن، از `plugins/BazzarMC/ranks.yml` استفاده می‌گردد)

### چه چیزی در نسخهٔ ۱.۸.۴ عوض شده است؟

* **رفع مشکل کلیک و دریافت در منوی کدهای خرید (`/mclink gui`)** — هنگام باز کردن یا ورق‌زدن منوی دریافت، رویداد بسته شدن اینونتوری قبلی فهرست آیتم‌ها را پاک می‌کرد و کلیک روی آیتم‌ها پاداشی نمی‌داد؛ این ساختار با کلاس اختصاصی `ClaimHolder` بازنویسی شد و داده‌های تحویل مستقیم درون اینونتوری منو نگهداری می‌شوند.
* **شناسایی و تطبیق گستردهٔ محصولات و دستورها** — تطبیق دقیق و فازی کلیدها و نام محصولات ووکامرس (حذف فاصله‌ها و علامت‌ها)، پشتیبانی از تعریف دستورها به‌صورت فهرست در زیر کلید محصول، دستور تکی، و پشتیبانی از نام‌های مختلف دستورهای کنسول (`commands`/`command`/`cmd`/`console-commands`) و بازیکن (`player-commands`/`player_command`).
* **ارتقای موتور پاداش و متغیرها (`RewardEngine`)** — پشتیبانی کامل از متغیرهای `{player_name}`، `{username}`، `{user}`، `{name}`، `{uuid_dashed}`، `{product_name}`، `{qty}`، `{quantity}`، `{order_id}`، `{purchase_code}` و `{role}`؛ اجرای مطمئن و همگام دستورها در رشتهٔ اصلی سرور و بازخورد شفاف با کلید پیشنهادی در صورت تعریف‌نشدن محصول در `config.yml`.

---

### چه چیزی در نسخهٔ ۱.۸.۳ عوض شده است؟

* **jar تغییر عملکردی ندارد** — فقط هم‌نسخهٔ افزونهٔ وردپرس شد؛ `config.yml`، دستورهای بازی و مسیرهای REST همان‌ها هستند و فایل قبلی شما دست‌نخورده کار می‌کند.
* سمت وردپرس در این نسخه **مشکل خاموش/روشن‌نشدن کلیدهای پنل** حل شد: ظاهر سوییچ‌ها پیش‌تر به کلاس سمت سرور چسبیده بود و با کلیک تغییر نمی‌کرد؛ اکنون وضعیت سوییچ از خودِ چک‌باکس خوانده می‌شود و همهٔ کلیدها در همهٔ تب‌ها درست ذخیره می‌شوند. پس از به‌روزرسانی، صفحهٔ تنظیمات را یک‌بار با **Ctrl+F5** تازه کنید.

---

### چه چیزی در نسخهٔ ۱.۸.۲ عوض شده است؟

* **بدون تغییر در خودِ jar** — ساختار `config.yml`، دستورهای بازی و مسیرهای REST همان‌ها هستند؛ فقط شمارهٔ نسخه هم‌نسخهٔ افزونهٔ وردپرس شد. فایل `config.yml` قبلی شما دست‌نخورده کار می‌کند.
* **سمت وردپرس، اتصال سرور شما از این نسخه «دیده» می‌شود:** هر درخواست تأییدشدهٔ پلاگین (راه‌اندازی سرور، `bmc ping`، `POST /link/redeem`، `POST /deliveries/*` و بقیه) در پنل ثبت می‌شود — زمان، آی‌پی، مسیر و تعداد موفق‌ها به‌همراه آخرین تلاش ناموفق و دلیل آن. مرحلهٔ «اتصال سرور بازی» در فهرست راه‌اندازی پیشخوان، ردیف «وضعیت اتصال سرور بازی» در تب *اتصال سرور* و بررسی سلامت همه از همین دادهٔ واقعی سبز/قرمز می‌شوند.
* **سریع‌ترین راه تست:** در کنسول سرور `bmc ping` بزنید (یا `mclink status` از سمت بازیکن). اگر پنل pill قرمز «توکن رد می‌شود» نشان داد، مقدار `api.token` در `config.yml` را با توکن تب *اتصال سرور* یکی کنید و `bmc reload` بزنید.
* **ثبت، حداکثر یک بار در دقیقه** انجام می‌شود تا نوشتن اضافی روی دیتابیس سایت نیفتد؛ وضعیت همیشه تازه است.
* سمت وردپرس هم در همین نسخه: رفع صفحهٔ «ویرایش/جزئیات حساب کاربری» و بقیهٔ بخش‌ها، باقی‌ماندن و درست‌کارکردن «تیکت‌ها» و «اتصال ماینکرافت»، کارت حساب کاربری **وابسته به بخش با محتوای متناسب** (پیش‌فرض «هوشمند» + «درج خودکار» روشن) و رفع خطای دکمهٔ **پاک‌سازی لاگ‌ها**.

---

### چه چیزی در نسخهٔ ۱.۸.۱ عوض شده است؟

* **رفع کامل مشکل «درخواست کد ایمیل» در بازی** — ریشهٔ مشکل سمت وردپرس بود: هنگام ساخت متن ایمیل، شیء کاربر وارد `str_replace` می‌شد و یک **خطای مهلک PHP** می‌ساخت؛ در نتیجه REST با خطای ۵۰۰ پاسخ می‌داد و بازیکن فقط پیام «مشکل دارد» می‌دید. اکنون ساخت متن ایمیل ایمن است و درخواست کد به‌درستی پاسخ می‌گیرد.
* **نمایش کد در چت وقتی ایمیل نرسد** — اگر سایت نتواند ایمیل بفرستد ولی کانال **داشبورد** روشن باشد، کد در پاسخ API برمی‌گردد و پلاگین همان لحظه آن را در چت چاپ می‌کند:

```yaml
messages:
  otp-code-inline: "&eEmail delivery was not available on the site, so your code is shown here: &f{code}"
  otp-code-command: "&7Run: &f{cmd} verify {code}"
```

  `{cmd}` به‌طور خودکار با دستور پلاگین (پیش‌فرض `/mclink`) جایگزین می‌شود. اگر ایمیل سالم ارسال شود، همان پیام قبلی «کد ارسال شد» نمایش داده می‌شود.
* **بدون تغییر در ساختار `config.yml`** — فقط دو کلید پیام تازه اضافه شده است؛ فایل قبلی شما دست‌نخورده کار می‌کند (برای گرفتن متن پیش‌فرض تازه، `bmc defaults` یا حذف و رستارت).
* سمت وردپرس هم در همین نسخه: کارت حساب کاربری **فقط در داشبورد**، رفع صفحه‌های «جزئیات حساب/اتصال ماینکرافت/تیکت‌ها»، بهبود خودکار پس از به‌روزرسانی (خاموش‌شدن حالت ایمن خودکار) و پیش‌فرض خاموش برای «حالت سخت‌گیرانهٔ کانال ایمیل».

---

### چه چیزی در نسخهٔ ۱.۸.۰ عوض شده است؟

* **`config.yml` کاملاً انگلیسی** — همهٔ کلیدها، توضیح‌ها و پیام‌های پلاگین (به‌همراه `plugin.yml`) انگلیسی شدند تا فایل تنظیمات در هر سروری بدون مشکل انکودینگ خوانده و ویرایش شود. ساختار کلیدها نسبت به ۱.۷.۰ عوض نشده؛ پس تنظیم‌های قبلی را مستقیم می‌توانید منتقل کنید (فقط متن توضیح‌ها انگلیسی شده است).
* **دستور یکپارچهٔ تأیید کد** — `/mclink verify <code>` یک دستور برای **همهٔ کدها** است؛ فرقی نمی‌کند کد از **ایمیل** آمده باشد یا از **پنل کاربری سایت**. شکل کوتاه `/mclink <code>` هم همان کار را می‌کند (`verify.allow-short-form: true`). دستور قدیمی `/mclink mail <email> <code>` هم برای سازگاری باقی مانده است.
* **رفع مشکل درخواست کد ایمیل** — سمت وردپرس، ارسال ایمیل با هدرهای کامل (`From`/`Reply-To`/`Content-Type`)، کدگذاری MIME نام و موضوع فارسی، آدرس/نام فرستندهٔ قابل تنظیم و ثبت دلیل واقعی شکست در لاگ اصلاح شد؛ در حالت `verify.strict-mode: true` اگر کانال ایمیل خاموش باشد، به‌جای سکوت، خطای روشن برگردانده می‌شود.
* **محصول لزوماً رنک نیست** — نمونهٔ آمادهٔ `config.yml` و متن‌های داخلی به‌صورت عمومی (آیتم، کیت، پول یا امتیاز درون‌بازی، کلید صندوقچه، دسترسی خاص، رنک یا دستورهای دلخواه) بازنویسی شد.
* **موتور پاداش عمومی (`RewardEngine`)** — هر خرید می‌تواند هر ترکیبی از این پاداش‌ها را داشته باشد: آیتم، **رنک**، **امتیاز (`points`)**، **پول (`money`)**، **اعلان عمومی (`broadcast`)**، دستورهای کنسول و دستورهای سمت بازیکن.
* **رنک حتی بدون تعریف محصول** — اگر کلید/نام/شناسهٔ محصول با یکی از رنک‌های `ranks.list` مطابقت کند، رنک با دستور `rewards.rank-command` اعطا می‌شود؛ حتی اگر در بخش `products` هیچ تعریفی نباشد (`rewards.unknown-rank: true`).
* **ردیف‌ها دیگر بی‌دلیل در صف نمی‌مانند** — هر پاداشی که اعطا شود، با `delivery.mark-granted-as-delivered: true` ردیف «تحویل‌شده» علامت می‌خورد و آیتم کاغذی از منوی دریافت حذف می‌گردد.
* **گزارش دلیل تحویل‌نشدن به سایت** (`delivery.report-to-site`) — با `POST /deliveries/report`؛ مدیر در پنل (تب *تحویل‌ها* ← ستون «گزارش») و کاربر در *تحویل‌های من* دلیل را می‌بیند و ردیف‌های گیرکرده در فیلتر «گیرکرده» فهرست می‌شوند.
* **خلاصهٔ پاداش** — پس از تحویل موفق، خلاصهٔ کوتاه (مثل «رنک VIP + ۲ دستور + ۶۴ آیتم») با `POST /deliveries/mark` و پارامتر `note` به سایت فرستاده می‌شود.
* **دستورهای جدید مدیر**: `/bmc rewards` و `/bmc reward <بازیکن> <کلید> [تعداد]`.
* **نمونهٔ آمادهٔ `config.yml` در پنل وردپرس** — تب *محصولات و نقش‌ها* برای هر محصول همگام‌شده یک بلوک YAML می‌سازد که با یک کلیک کپی و در `products` جای‌گذاری می‌شود.

---

## نصب

```bash
# ۱) کپی در پوشهٔ plugins
cp ../dist/BazzarMC-1.8.3.jar /path/to/server/plugins/

# ۲) اجرای سرور (config.yml ساخته می‌شود)
# ۳) ویرایش plugins/BazzarMC/config.yml
# ۴) در کنسول سرور
bmc reload
bmc ping
bmc syncranks      # ارسال تعریف رنک‌ها به سایت
```

در `config.yml` فقط دو مقدار الزامی است:

```yaml
api:
  base-url: "https://yoursite.com"    # no trailing slash
  token: "TOKEN_FROM_WP_PANEL"        # WP Admin -> BazzarMc -> Server Connection
```

> از نسخهٔ ۱.۸.۰ همهٔ توضیح‌های `config.yml` **انگلیسی** است. مقدارها (مثل آدرس سایت و توکن) همان‌طور که هستند نوشته می‌شوند و زبان آن‌ها مهم نیست؛ فقط بخش `products` ممکن است نام فارسی محصول شما را داشته باشد (تطبیق با نام محصول، نام‌محور هم هست).

---

## دستورها

> **نام دستورهای پلاگین کاملاً قابل تغییر است.** در `config.yml` می‌توانید `command.name` (دستور بازیکن) و `command.admin-name` (دستور مدیر) را به هر چیزی که می‌خواهید تغییر دهید و فهرست نام‌های جایگزین (`command.aliases` / `command.admin-aliases`) را هم آزادانه تعیین کنید. دستورهای سفارشی در زمان اجرا ثبت می‌شوند؛ پس از تغییر، سرور را رستارت کنید (یا `bmc reload`). در جدول‌های زیر از نام پیش‌فرض استفاده شده است.

### بازیکنان — `/mclink` (دسترسی: `bazzarmc.use`، پیش‌فرض: همه)

| دستور | کار |
|---|---|
| `/mclink verify <code>` | **تأیید هر کدی** — کد پنل کاربری سایت یا رمز ایمیلی (جدید در ۱.۸.۰) |
| `/mclink <code>` | شکل کوتاه همان دستور تأیید (قابل خاموش‌کردن با `verify.allow-short-form`) |
| `/mclink mail <email>` | ارسال رمز از راه ایمیل |
| `/mclink mail <email> <code>` | تأیید رمز ایمیلی و اتصال (سازگار با نسخهٔ قدیمی) |
| `/mclink status` | وضعیت اتصال + تعداد آیتم‌های در انتظار |
| `/mclink claim` | دریافت خودکار همهٔ آیتم‌های در انتظار در اینونتوری |
| `/mclink gui` | **منوی دریافت آیتم‌ها** — هر خرید یک کاغذ با نام «کد خرید» است و با کلیک تحویل می‌گیرد |
| `/mclink rank` | **رنک فعلی، سطح و زمان باقی‌مانده** (از دید پلاگین) |
| `/mclink help` | راهنما |

نام‌های جایگزین: `/storelink`، `/slink`، `/bazzar`، `/link`.

### مدیران — `/bmc` (دسترسی: `bazzarmc.admin`، پیش‌فرض: op)

| دستور | کار |
|---|---|
| `/bmc version` | نسخه و وضعیت پیکربندی (تحویل، GUI، رنک‌ها) |
| `/bmc reload` | بارگذاری مجدد `config.yml` و راه‌اندازی دوبارهٔ زمان‌بندها |
| `/bmc ping` | تست اتصال به سایت (نسخهٔ افزونه و زمان سرور) |
| `/bmc token` | نمایش توکن و آدرس API فعلی |
| `/bmc deliveries <بازیکن>` | فهرست تحویل‌های در انتظار یک بازیکن |
| `/bmc unlink <بازیکن>` | لغو اتصال یک بازیکن |
| `/bmc ranks` | فهرست رنک‌های تعریف‌شده، وضعیت همگام‌سازی و زمان آخرین گزارش |
| `/bmc syncranks` | ارسال فوری تعریف رنک‌ها به سایت (`ranks/definitions`) |
| `/bmc report <بازیکن>` | ارسال فوری گزارش زمان باقی‌ماندهٔ رنک یک بازیکن (`ranks/report`) |
| `/bmc gui [بازیکن]` | بازکردن منوی دریافت برای خود یا یک بازیکن آنلاین |
| `/bmc products` | دریافت فهرست محصولات سایت با کلید تحویل، نقش، اعتبار و سطح رنک |
| `/bmc rewards` | نمایش پیکربندی فعلی پاداش‌ها: شمار محصولات تعریف‌شده، رنک‌ها، دستورهای `rewards.*` و وضعیت `mark-granted-as-delivered` / `report-to-site` |
| `/bmc reward <بازیکن> <کلید> [تعداد]` | اجرای دستی پاداش یک کلید محصول برای یک بازیکن آنلاین (بدون صف سایت) — برای تست `config.yml` |

---

## تنظیمات

> از نسخهٔ **۱.۸.۰** فایل `config.yml` کاملاً **انگلیسی** است (کلیدها، توضیح‌ها و پیام‌های پیش‌فرض).
> ساختار کلیدها نسبت به ۱.۷.۰ تغییر نکرده؛ بنابراین مقدارهای فایل قبلی را می‌توانید مستقیم منتقل کنید.
> متن پیام‌ها (`&a...`) و نام‌های محصول را می‌توانید فارسی بنویسید — فقط **کلیدها و توضیح‌ها** انگلیسی‌اند.

```yaml
api:
  base-url: "https://yoursite.com"       # no trailing slash
  token: "CHANGE_ME"                     # WP Admin -> BazzarMc -> Server Connection
  endpoint: "/wp-json/bazzarmc/v1"       # normally no need to change
  timeout-seconds: 10
  verify-ssl: true                       # false = accept self-signed certificate (matches ssl_verify in WP)

store:
  name: "My Store"                       # {store}
  url: ""                                # {store_url} - empty means the same as api.base-url
  ip: ""                                 # {ip} - empty means auto-detect from server.properties
  server-name: ""                        # {server}

command:
  name: "mclink"                         # player command name (fully customizable)
  aliases: [bazzar, storelink, slink, link]
  admin-name: "bmc"                      # admin command name
  admin-aliases: [bazzarmcadmin, bmcadmin]

verify:                                  # NEW in 1.8.0 - one command for every code
  strict-mode: true                      # true = clear error when the requested channel is disabled
  allow-short-form: true                 # /mclink <code> works like /mclink verify <code>
  accept-email-codes: true               # /mclink verify also accepts emailed one-time codes

link:
  broadcast: true                        # announce the link in the public chat
  broadcast-message: "&a{player} &7linked their account to the store!"
  request-cooldown: 60                   # seconds between two code requests

delivery:
  enabled: true
  poll-seconds: 45                       # periodic check for online players
  on-join: true                          # check when a player joins
  join-delay-seconds: 5                  # delay after join until the player is fully loaded
  unknown-product-commands: []           # fallback commands for a product not defined here
  unknown-product-message: "&eYour purchase &f{product} &eis not defined on the server yet. An admin was notified."
  mark-granted-as-delivered: true        # mark the row delivered once any reward was granted
  mark-unknown-as-delivered: false       # false = keep it in the queue for the admin
  report-to-site: true                   # send the delivery reason/attempts back to the site
  success-message: "&a{product} was delivered with purchase code &f{code}&a."

rewards:                                 # global reward commands (overridable per product)
  rank-command: "lp user {player} parent add {rank_role}"
  rank-temp-command: "lp user {player} parent addtemp {rank_role} {rank_days}d accumulate"
  points-command: "points give {player} {points}"
  money-command: "eco give {player} {money}"
  broadcast-command: ""                  # empty = plain Bukkit broadcast
  unknown-rank: true                     # grant a rank matched from ranks.list even without a products entry
  log-to-console: true

ranks:
  enabled: true
  sync-to-site: true                     # send rank definitions and remaining time to WordPress
  sync-interval-minutes: 60              # how often definitions are sent
  report-interval-minutes: 60            # how often remaining time of online players is sent
  prevent-downgrade: true                # do not deliver a lower rank to a higher-rank owner
  downgrade-blocked-message: "&cYou already hold a higher rank; this item is not delivered."
  list:                                  # the single source of rank definitions (here, not the site)
    vip:                                 # key = product key in products: or the WP delivery id
      name: "VIP"
      level: 1                           # higher number = higher rank
      role: "vip"                        # LuckPerms group name
      days: 30                           # 0 = permanent
    legend:
      name: "Legend"
      level: 3
      role: "legend"
      days: 0

gui:
  enabled: true
  title: "&8&lYour purchases"
  size: 54                               # 18 to 54 (multiple of 9)
  code-material: PAPER                   # the purchase-code item
  item-name: "&e&lPurchase code: &f{code}"
  item-lore:
    - "&7Product: &f{product}"
    - "&7Item: &f{item} &7x&f{amount}"
    - "&7Order: &f{order_number}"
    - "&aClick this item to claim it."
  filler-material: BLACK_STAINED_GLASS_PANE
  previous-material: ARROW
  next-material: ARROW
  refresh-material: COMPARATOR
  page-material: BOOK
  close-material: BARRIER
  reminder-enabled: true                 # remind about unclaimed items
  reminder-interval-seconds: 300
  reminder-message: "&eYou have &f{count} &eunclaimed item(s) in &f{store} &e(code &f{code}&e). To claim: &f{gui_cmd}"
  remind-when-blocked: true              # remind when the inventory is full
```

---

## تعریف محصولات

> **محصول لزوماً رنک نیست.** هر چیزی که در فروشگاه می‌فروشید می‌تواند یک محصول باشد: آیتم یا کیت درون‌بازی، پول یا امتیاز، کلید صندوقچه، دسترسی خاص، رنک، یا مجموعه‌ای از دستورهای کنسول.
> در `config.yml` فقط تعیین می‌کنید **پاداش** آن خرید چیست (کلیدهای `material`/`amount`/`name`/`lore`/`commands`/`player-commands`/`points`/`money`/`broadcast`/`rank`)؛
> اگر محصول شما رنک نیست، کلید `rank` را ننویسید و پاداش دلخواهتان را تعریف کنید.

کلید هر محصول در `config.yml` با «شناسهٔ تحویل» محصول در پنل وردپرس (تب *محصولات و نقش‌ها*) مطابقت داده می‌شود.
تطبیق به این ترتیب انجام می‌شود (اولین موردِ منطبق، برنده است):

| ترتیب | کلید | توضیح |
|---|---|---|
| ۱ | **شناسهٔ تحویل پنل** (`key`) | همان چیزی که مدیر در تب محصولات انتخاب کرده: نام / ID / نامک / SKU / متن دلخواه |
| ۲ | **نام محصول** (`item`) | نام محصول ووکامرس با حروف کوچک |
| ۳ | **شناسهٔ نسخه** (`variation_id`) | برای محصولات متغیر |
| ۴ | **شناسهٔ محصول** (`product_id`) | ID عددی محصول در ووکامرس |
| ۵ | **نامک** (`slug`) | مثل `gold_pack_2` |
| ۶ | **کد کالا** (`sku`) | اگر در ووکامرس پر شده باشد |

> مثال: محصولی با نام نمایشی «کوین ۱۰۰» و نامک `gold_pack_2` ساخته‌اید؛ در `config.yml` می‌توانید کلید را `"کوین ۱۰۰"`، `"gold_pack_2"`، `"123"` (شناسه) یا هر متن دلخواهی که در پنل تعیین کرده‌اید بگذارید.

```yaml
products:
  # 0) a non-rank product: in-game money + a crate key (no "rank" key at all)
  "gold pack":
    money: 25000
    material: TRIPWIRE_HOOK
    amount: 1
    name: "&6Crate Key"
    message: "&625000 coins and a crate key were added to your account."

  # ۱) فقط دستور کنسول (مثلاً LuckPerms)
  "vip":
    message: "&a&lرتبهٔ VIP &7برای شما فعال شد!"
    commands:
      - "lp user {player} parent add temporary vip {days}d"

  # ۲) فقط آیتم
  "64 diamond":
    material: DIAMOND
    amount: 64
    message: "&b۶۴ الماس &7دریافت کردید!"

  # ۳) آیتم با نام و lore سفارشی
  "crate key":
    material: TRIPWIRE_HOOK
    amount: 3
    name: "&6کلید صندوقچه"
    lore:
      - "&7با این کلید صندوقچهٔ شانس را باز کنید"

  # ۴) ترکیبی
  "starter kit":
    material: IRON_SWORD
    amount: 1
    name: "&fشمشیر تازه‌کار"
    commands:
      - "give {player} bread 16"
      - "eco give {player} 5000"

  # ۵) تطبیق با شناسهٔ محصول ووکامرس
  "355":
    material: NETHERITE_INGOT
    amount: 1

  # ۶) نام محصول فارسی هم کار می‌کند (مقدار item در API دقیقاً نام محصول ووکامرس با حروف کوچک است)
  "رتبه ویژه":
    message: "&6رتبهٔ ویژه فعال شد!"
    commands:
      - "lp user {player} parent add special"
```

> مقدار `item` در پاسخ `GET /deliveries` برابر **نام محصول ووکامرس با حروف کوچک** است
> (در PHP با `mb_strtolower`)، پس برای محصولات فارسی همان نام را کلید کنید.

### همهٔ کلیدهای پاداش یک محصول (جدید در ۱.۷.۰)

```yaml
products:
  "legend pack":
    rank: legend                # اختیاری — کلید رنک از ranks.list (اگر دستور اختصاصی نباشد، rewards.rank-command اجرا می‌شود)
    material: NETHER_STAR       # اختیاری — آیتم فیزیکی
    amount: 1                   # تعداد آیتم (× تعداد سفارش)
    name: "&6ستارهٔ لجند"      # نام سفارشی آیتم
    lore:                       # توضیحات آیتم
      - "&7پاداش خرید لجند"
    commands:                   # دستورهای کنسول (سرور)
      - "lp user {player} parent add temp legend {days}d"
    player-commands:            # دستورهایی که از طرف خود بازیکن اجرا می‌شوند (بدون اسلش)
      - "kit legend"
    points: 250                 # امتیاز — با rewards.points-command اعطا می‌شود
    money: 5000                 # پول — با rewards.money-command اعطا می‌شود
    broadcast: "&6{player} &7پک لجند را خرید!"   # اعلان عمومی (یا دستور، اگر rewards.broadcast-command پر باشد)
    message: "&6پک لجند فعال شد — تا &f{expire_date}&6."
    mark-delivered: true        # false = ردیف در صف سایت بماند (تحویل دستی توسط مدیر)
```

| کلید | نوع | پیش‌فرض | توضیح |
|---|---|---|---|
| `rank` | string | – | کلید رنک در `ranks.list`؛ اگر محصول خودش `commands` رنک نداشته باشد، از `rewards.rank-command` استفاده می‌شود |
| `material` / `amount` / `name` / `lore` | – | – | آیتم فیزیکی (همان رفتار نسخه‌های قبل) |
| `commands` | list | `[]` | دستورهای کنسول با متغیرها |
| `player-commands` | list | `[]` | **جدید** — دستور از طرف بازیکن |
| `points` | int | `0` | **جدید** — امتیاز؛ دستور آن `products.<key>.points-command` یا `rewards.points-command` است |
| `money` | int | `0` | **جدید** — پول؛ دستور آن `products.<key>.money-command` یا `rewards.money-command` |
| `points-command` / `money-command` | string | – | **جدید** — بازنویسی دستور همان محصول |
| `broadcast` | string | – | **جدید** — اعلان عمومی؛ متغیر `{broadcast}` = متن رنگی‌شده |
| `message` | string | – | پیام خصوصی به بازیکن |
| `mark-delivered` | bool | `true` | **جدید** — `false` = ردیف در صف بماند (مثلاً تحویل دستی) |

> `points` و `money` در **تعداد سفارش** ضرب می‌شوند: خرید ۲ عدد «پک ۲۵۰ امتیازی» = ۵۰۰ امتیاز.

### بخش عمومی `rewards:` (جدید در ۱.۷.۰)

این بخش، رفتار پیش‌فرض پاداش‌ها را برای **همهٔ محصولات** تعیین می‌کند:

```yaml
rewards:
  rank-command: "lp user {player} parent add {rank_role}"      # دستور اعطای رنک
  rank-temp-command: ""                                        # رنک مدت‌دار؛ خالی = همان rank-command
  points-command: "points give {player} {points}"              # دستور اعطای امتیاز
  money-command: "eco give {player} {money}"                   # دستور اعطای پول
  broadcast-command: ""                                        # خالی = پخش مستقیم در چت سرور
  unknown-rank: true                                            # برای محصول تعریف‌نشده هم رنک اعطا شود؟
  log-to-console: true                                          # ثبت نتیجهٔ هر بررسی در کنسول
```

و در بخش `delivery:`:

```yaml
delivery:
  mark-granted-as-delivered: true   # هر پاداش اعطاشده ⇒ ردیف «تحویل‌شده» (رفع گیرماندن در صف)
  mark-unknown-as-delivered: false  # محصول ناشناخته بدون هیچ پاداشی هم «تحویل‌شده» شود؟
  report-to-site: true              # ارسال دلیل تحویل‌نشدن به سایت (POST /deliveries/report)
```

### نمونهٔ آماده از پنل وردپرس

در پیشخوان وردپرس ← **BazzarMc ← محصولات و نقش‌ها**، پایین صفحه دو بلوک YAML آماده هست:

1. **«نمونهٔ پاداش برای محصولات همگام‌شده»** — برای هر محصول انتخاب‌شده یک بلوک `products` با کلید درست، `rank`، دستور LuckPerms، `points`، `money`، `broadcast` و `mark-delivered: true` می‌سازد.
2. **«تنظیمات عمومی پاداش‌ها»** — همان بلوک `rewards:` بالا با مقدارهای فعلی پنل شما.

هر دو با دکمهٔ «کپی» برداشته و در `plugins/BazzarMC/config.yml` جای‌گذاری می‌شوند؛ سپس `bmc reload`.

### متغیرهای قابل استفاده در `commands` و `message`

| متغیر | مقدار |
|---|---|
| `{player}` | نام بازیکن |
| `{uuid}` | UUID بدون خط تیره |
| `{item}` | کلید/نام محصول |
| `{amount}` | تعداد سفارش‌داده‌شده |
| `{order}` | شناسهٔ سفارش ووکامرس |
| `{order_number}` | شمارهٔ سفارش (همان چیزی که کاربر در سایت می‌بیند) |
| `{code}` | **کد خرید** (پیش‌فرض: `شمارهٔ سفارش-شناسهٔ ردیف`، مثل `1042-7`) |
| `{product}` | نام نمایشی محصول (یا کلید/آیتم اگر نام خالی بود) |
| `{product_id}` | شناسهٔ محصول |
| `{variation_id}` | شناسهٔ نسخه |
| `{rank_level}` | سطح رنک محصول (۰ اگر رنک نباشد) |
| `{days}` / `{rank_days}` | مدت اعتبار رنک از `ranks.list` (۰ اگر رنک نباشد یا دائمی باشد) — مناسب `lp user {player} parent add temporary {rank_role} {days}d` |
| `{rank}` / `{rank_name}` | نام رنک منطبق‌شده |
| `{rank_role}` | گروه (`role`) رنک منطبق‌شده |
| `{delivery_id}` | شناسهٔ ردیف در صف تحویل |
| `{key}` | شناسهٔ تحویل تعیین‌شده در پنل وردپرس |
| `{slug}` | نامک محصول |
| `{sku}` | کد کالای محصول |
| `{store}` | نام فروشگاه (`store.name`) |
| `{store_url}` | آدرس فروشگاه (`store.url`) |
| `{ip}` | آی‌پی/دامنهٔ سرور (`store.ip`) |
| `{server}` | نام سرور (`store.server-name`) |
| `{cmd}` | نام دستور بازیکن با اسلش (مثل `/mclink`) |
| `{gui_cmd}` | دستور بازکردن منوی دریافت (مثل `/mclink gui`) |
| `{admin_cmd}` | نام دستور مدیر با اسلش (مثل `/bmc`) |
| `{plugin_version}` | نسخهٔ پلاگین |
| `{rank_key}` | کلید رنک منطبق‌شده در `ranks.list` |
| `{seconds}` | مدت اعتبار رنک به ثانیه (`days` × ۸۶۴۰۰ × تعداد سفارش) |
| `{expire_at}` | زمان انقضای رنک (unix) — `0` اگر دائمی/بدون رنک |
| `{expire_date}` | تاریخ انقضای رنک به شکل `yyyy/MM/dd HH:mm` |
| `{points}` | مقدار امتیاز همان محصول (فقط در دستور امتیاز) |
| `{money}` | مقدار پول همان محصول (فقط در دستور پول) |
| `{broadcast}` | متن رنگی‌شدهٔ `broadcast` همان محصول (فقط در `rewards.broadcast-command`) |
| `{suggest_key}` | کلید پیشنهادی برای تعریف محصول در `config.yml` (فقط در `messages.reward-none`) |

> متغیرهای `{store}`، `{store_url}`، `{ip}`، `{server}`، `{cmd}`، `{gui_cmd}` و `{admin_cmd}` در **همهٔ پیام‌های `messages`** هم قابل استفاده هستند.

> **نکتهٔ تعداد**: `amount` آیتم = `amount` تنظیمات × `amount` سفارش. یعنی اگر کاربر ۲ عدد «کلید صندوقچه» بخرد و `amount: 3` باشد، ۶ کلید دریافت می‌کند. مدت رنک هم به همین شکل تجمیع می‌شود: خرید ۲ عدد «VIP یک‌ماهه» = ۶۰ روز.

### رنگ‌ها

علاوه بر کدهای کلاسیک (`&a`، `&l` و…)، کد رنگ هگز هم پشتیبانی می‌شود (MC 1.16+):

```yaml
message: "&#00ff88متن سبز نئونی!"
```

---

## منوی دریافت آیتم‌ها (GUI)

* با `/mclink gui` (یا `/bmc gui <بازیکن>` از طرف مدیر) یک منوی صندوقچه‌ای باز می‌شود.
* **هر خرید = یک آیتم کاغذی** که نامش **کد خرید** است (`gui.item-name` با متغیر `{code}`) و توضیحاتش محصول، آیتم، تعداد و شمارهٔ سفارش را نشان می‌دهد.
* با کلیک روی همان آیتم، **فقط همان خرید** بررسی می‌شود و هر پاداش تعریف‌شده (آیتم، رنک، امتیاز، پول، اعلان، دستور کنسول/بازیکن) اعطا می‌گردد؛ اگر پاداشی اعطا شده باشد، ردیف در سایت «تحویل‌شده» علامت می‌خورد، آیتم کاغذی از منو حذف می‌شود و خلاصهٔ پاداش در `note` سایت ثبت می‌گردد.
* اگر هیچ پاداشی اعطا نشود، آیتم کاغذی **حذف نمی‌شود** و دلیل آن با `POST /deliveries/report` به سایت فرستاده می‌شود؛ بازیکن پیام `messages.reward-none` را می‌گیرد که کلید پیشنهادی `config.yml` را هم نشان می‌دهد.
* اگر اینونتوری جای خالی نداشته باشد، پیام `messages.inventory-full` نمایش داده می‌شود و ردیف در صف می‌ماند.
* اگر بازیکن رنک بالاتری داشته باشد و خرید مربوط به رنک پایین‌تر باشد (`ranks.prevent-downgrade`)، تحویل متوقف و پیام `ranks.downgrade-blocked-message` ارسال می‌شود.
* منو صفحه‌بندی دارد (دکمه‌های صفحهٔ قبل/بعد، شمارهٔ صفحه، بازخوانی و بستن) و همهٔ مواد و متن‌هایش در `gui.*` قابل تغییر است.
* شناسهٔ هر ردیف در `PersistentDataContainer` آیتم ذخیره می‌شود؛ پس آیتم‌های تقلبیِ ساخته‌شده توسط بازیکن هیچ اثری ندارند و کلیک‌های متوالی سریع هم نادیده گرفته می‌شوند.

### یادآوری آیتم‌های تحویل‌گرفته‌نشده

با `gui.reminder-enabled` و `gui.reminder-interval-seconds` (پیش‌فرض ۵ دقیقه) به بازیکنانی که آیتم تحویل‌گرفته‌نشده دارند پیام `gui.reminder-message` ارسال می‌شود (با `{count}`، `{code}`، `{product}` و `{gui_cmd}`). وقتی منو باز باشد یا اینونتوری پر باشد هم یادآوری تکرار نمی‌شود/ارسال می‌شود (طبق `gui.remind-when-blocked`).

---

## رنک‌ها و همگام‌سازی با سایت

1. **تعریف رنک در `ranks.list`** (کلید، نام، `level`، `role`، `days`) — منبع اصلی، سمت ماینکرافت است.
2. پلاگین هر `ranks.sync-interval-minutes` دقیقه (و هنگام `bmc syncranks`) تعریف‌ها را با `POST /ranks/definitions` به سایت می‌فرستد؛ سایت از روی آن، سطح رنک هر محصول ووکامرس را می‌شناسد و **خرید رنک پایین‌تر را برای دارندهٔ رنک بالاتر مسدود** می‌کند.
3. هر `ranks.report-interval-minutes` دقیقه، **زمان باقی‌ماندهٔ رنک بازیکنان آنلاین** با `POST /ranks/report` به سایت ارسال می‌شود؛ سایت آن را در متای `bmc_role_grants` ذخیره می‌کند و رنک‌هایی که دیگر گزارش نشده‌اند بازپس گرفته می‌شوند.
4. منبع زمان باقی‌مانده: اگر **LuckPerms** نصب باشد، انقضای واقعی گروه (`group.<role>`) خوانده می‌شود؛ در غیر این صورت از فایل `plugins/BazzarMC/ranks.yml` استفاده می‌شود که هنگام تحویل هر رنک (و با `days` × تعداد) به‌روزرسانی می‌گردد. خرید مجدد، مدت را **تجمیع** می‌کند.
5. `/mclink rank` همین اطلاعات را به بازیکن نشان می‌دهد و `/bmc ranks` وضعیت همگام‌سازی را به مدیر.

> تطبیق رنک با محصول هم با همان شش کلید بخش محصولات انجام می‌شود؛ پس کلید `ranks.list` را مطابق کلید `products` (یا شناسهٔ تحویل پنل) بگذارید.

---

## نحوهٔ کار تحویل

1. زمان‌بند هر `poll-seconds` ثانیه و همچنین هنگام ورود هر بازیکن، فهرست تحویل‌های در انتظار را از `GET /deliveries?player=…` می‌گیرد.
2. برای هر ردیف، بخش متناظر در `products` پیدا می‌شود (شناسهٔ تحویل پنل → نام → شناسهٔ نسخه → شناسهٔ محصول → نامک → SKU).
3. اگر ردیف مربوط به رنک پایین‌تر از رنک فعلی بازیکن باشد، تحویل متوقف می‌شود (`ranks.prevent-downgrade`).
4. **`RewardEngine.grant()`** همهٔ پاداش‌ها را به‌ترتیب اجرا می‌کند: آیتم‌ها (سرریز روی زمین می‌افتد) → رنک → `commands` و `player-commands` → دستور رنک → `points` → `money` → `broadcast`. سپس `message` محصول و `delivery.success-message` ارسال می‌گردد.
5. اگر ردیف یک رنک باشد (چه از `products.<key>.rank`، چه از تطبیق با `ranks.list` وقتی محصول تعریف نشده است و `rewards.unknown-rank: true`)، مدت اعتبار در `ranks.yml` ثبت/تجمیع و گزارش به سایت ارسال می‌شود.
6. **اگر پاداشی اعطا شده باشد** و `delivery.mark-granted-as-delivered: true` (یا `mark-delivered: true` خود محصول)، `POST /deliveries/mark` با `note` = خلاصهٔ پاداش فراخوانی می‌شود تا ردیف «تحویل‌شده» شود و از صف و منوی دریافت خارج گردد.
7. **اگر هیچ پاداشی اعطا نشده باشد**، `POST /deliveries/report` دلیل را به سایت می‌فرستد (ستون «گزارش» در پنل و «تحویل‌های من»)، `attempts` یکی زیاد می‌شود، رویداد `warning` در لاگ افزونه ثبت می‌گردد و ردیف در صف می‌ماند. بازیکن پیام `messages.reward-none` را با کلید پیشنهادی می‌گیرد.
8. اگر اینونتوری **هیچ جای خالی** نداشته باشد، هیچ پاداشی اجرا نمی‌شود و ردیف در صف می‌ماند؛ دلیل «اینونتوری پر» گزارش می‌شود و بازیکن می‌تواند بعداً `/mclink claim` یا `/mclink gui` بزند.
9. اگر محصول در `products` تعریف نشده باشد، `delivery.unknown-product-commands` و (در صورت تطبیق رنک) دستور رنک اجرا می‌شود؛ سپس طبق `mark-granted-as-delivered` / `mark-unknown-as-delivered` تصمیم گرفته می‌شود ردیف تحویل‌شده علامت بخورد یا در صف بماند.

همهٔ درخواست‌های شبکه در ترد جداگانه اجرا می‌شوند (`runTaskTimerAsynchronously` + `CompletableFuture`) و هیچ‌کدام سرور را بلوکه نمی‌کنند.

---

## ساخت از سورس

پیش‌نیاز: JDK 17 (یا 11) و Maven 3.6+.

```bash
mvn clean package
# → target/BazzarMC-1.8.3.jar
```

با JDK/Maven دانلودشده در مسیر دیگر:

```bash
JAVA_HOME=/path/to/jdk17 /path/to/maven/bin/mvn clean package
```

---

## ساختار سورس

```text
src/main/
├── resources/
│   ├── plugin.yml                     تعریف پلاگین، دستورها و دسترسی‌ها
│   └── config.yml                     تنظیمات پیش‌فرض (فارسی)
└── java/studio/obsifox/bazzarmc/
    ├── BazzarMC.java                  کلاس اصلی، بارگذاری، پیام‌رسانی
    ├── ApiClient.java                 کلاینت ناهمگام REST (HttpURLConnection + Gson)
    ├── DeliveryService.java           زمان‌بندی، تطبیق محصولات، فراخوانی موتور پاداش و گزارش به سایت
    ├── RewardEngine.java              موتور پاداش: آیتم/رنک/امتیاز/پول/اعلان/دستورها، خلاصه و دلیل (۱.۷.۰)
    ├── ClaimGui.java                  منوی دریافت آیتم‌ها (کاغذ با نام کد خرید) + یادآوری
    ├── RankSyncService.java           تعریف رنک‌ها، ثبت/تجمیع اعتبار و گزارش به سایت
    ├── command/LinkCommand.java       /mclink
    ├── command/AdminCommand.java      /bmc
    ├── model/ApiResult.java           پوشش پاسخ JSON
    ├── model/Delivery.java            یک ردیف صف تحویل (شامل کد خرید)
    ├── model/Rank.java                یک تعریف رنک
    ├── util/LuckPermsProbe.java       خواندن انقضای گروه‌ها از LuckPerms (بازتابی، بدون وابستگی)
    └── util/Msg.java                  رنگ‌ها (& و hex) و جایگزینی متغیرها
```

---

## عیب‌یابی

| نشانه | علت / راه‌حل |
|---|---|
| `پلاگین پیکربندی نشده است` | `api.token` هنوز `CHANGE_ME` است یا `api.base-url` خالی است. |
| `سرور خطا داد (HTTP 403)` | توکن اشتباه است؛ از پنل وردپرس کپی مجدد کنید (`bmc token` برای مقایسه). |
| `HTTP 404` | مسیر REST درست نیست؛ سایت باید پیوندهای یکتا (Permalinks) را روی حالت غیر«ساده» داشته باشد. |
| `خطای ارتباطی: SSLHandshakeException` | گواهی SSL سایت معتبر نیست (یا زنجیرهٔ گواهی ناقص). گواهی را اصلاح کنید یا موقتاً `api.verify-ssl: false`. |
| آیتم تحویل می‌شود ولی در صف می‌ماند | فراخوانی `deliveries/mark` ناموفق بوده؛ لاگ کنسول و تب *صف تحویل‌ها* در پنل را ببینید. |
| محصول ناشناخته | کلید بخش `products` با نام محصول ووکامرس (حروف کوچک) یا شناسهٔ محصول یکی نیست. با `/bmc deliveries <بازیکن>` مقدار `item` را ببینید و همان را کلید کنید، یا از نمونهٔ آمادهٔ تب *محصولات و نقش‌ها* در پنل استفاده کنید. |
| خرید تحویل نمی‌شود و ردیف پاک نمی‌شود | هیچ پاداشی برای آن محصول تعریف نشده است. دلیل دقیق در پنل ← *تحویل‌ها* ← ستون «گزارش» و در *تحویل‌های من* کاربر دیده می‌شود (نیازمند `delivery.report-to-site: true`). اگر محصول یک رنک است، مطمئن شوید کلیدش با `ranks.list` مطابقت دارد یا `rewards.unknown-rank: true` است. |
| ردیف‌ها در صف گیر کرده‌اند | تب *تحویل‌ها* ← فیلتر **«گیرکرده»** (ردیف‌های در انتظار با تلاش ناموفق یا قدیمی‌تر از آستانهٔ پنل). پس از اصلاح `config.yml`، `bmc reload` بزنید و بازیکن دوباره `/mclink claim` یا کلیک در منو را انجام دهد؛ در پنل هم دکمهٔ «پاک‌کردن گزارش» شمار تلاش را صفر می‌کند. |
| رنک داده می‌شود ولی در صف می‌ماند | `delivery.mark-granted-as-delivered: false` یا `products.<key>.mark-delivered: false` است. |
| خلاصهٔ پاداش در پنل دیده نمی‌شود | افزونهٔ وردپرس باید ۱.۷.۰ یا بالاتر باشد (ستون `note` و `attempts` در همان نسخه اضافه شد). |
| `/mclink verify` شناخته نمی‌شود | jar باید نسخهٔ ۱.۸.۰ یا بالاتر باشد؛ پس از جایگزینی jar سرور را رستارت کنید یا `bmc reload` بزنید. |
| درخواست کد ایمیل خطا می‌دهد («مشکل دارد و ارسال نمی‌شود») | افزونهٔ وردپرس **و** jar باید **۱.۸.۱** باشند (خطای مهلک ساخت ایمیل در ۱.۸.۱ رفع شد). سپس در پنل: *تأیید و حساب کاربری* ← «ارسال ایمیل تست» و «حالت سخت‌گیرانهٔ کانال ایمیل» خاموش. اگر ایمیل نرسد، کانال **داشبورد** را روشن نگه دارید تا کد در چت چاپ شود. |
| کد تأیید در چت چاپ نمی‌شود | فقط وقتی چاپ می‌شود که پاسخ API شامل `code` باشد، یعنی ایمیل ارسال نشده و کانال داشبورد روشن است. کلیدهای `messages.otp-code-inline` و `messages.otp-code-command` در `config.yml` قابل ویرایش‌اند (jar ≥ ۱.۸.۱). |
| توضیح‌های `config.yml` هنوز فارسی است | فایل تنظیمات در نخستین اجرا ساخته می‌شود؛ برای گرفتن نسخهٔ انگلیسی ۱.۸.۰، از `plugins/BazzarMC/config.yml` پشتیبان بگیرید، آن را حذف کنید و سرور را رستارت کنید (یا `bmc defaults` را اجرا کنید) و سپس مقدارهای خود را برگردانید. |
| منوی GUI خالی باز می‌شود و بسته می‌شود | بازیکن آیتم تحویل‌گرفته‌نشده ندارد (`/bmc deliveries <بازیکن>`) یا `gui.enabled: false` است. |
| رنک‌ها در پنل وردپرس دیده نمی‌شوند | `ranks.list` خالی است یا `ranks.sync-to-site: false`؛ یک‌بار `bmc syncranks` بزنید و تب *محصولات و نقش‌ها* را ببینید. |
| زمان باقی‌ماندهٔ رنک در سایت به‌روز نمی‌شود | بازیکن باید آنلاین باشد تا گزارش شود (`ranks.report-interval-minutes`)؛ برای تست `bmc report <بازیکن>`. در لاگ سایت (تب ابزارها) رویداد `rank` ثبت می‌شود. |
| آیتم رنک پایین‌تر تحویل داده نمی‌شود | رفتار عمدی `ranks.prevent-downgrade` است؛ اگر لازم نیست آن را `false` کنید. |
| پیام‌ها رنگ ندارند | کدها را با `&` بنویسید و `bmc reload` بزنید. |
