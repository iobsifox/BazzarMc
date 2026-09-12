# BazzarMc — پلاگین سمت سرور ماینکرافت

پلاگین Spigot/Paper که فروشگاه ووکامرس (افزونهٔ **BazzarMc**) را به سرور ماینکرافت متصل می‌کند:
کد اتصال از پنل کاربری، اتصال با رمز یک‌بارمصرف پیامکی از داخل بازی، و تحویل خودکار آیتم‌ها/دستورها پس از پرداخت.

* خروجی آماده: `../dist/BazzarMC-1.0.0.jar`
* سازگاری: Spigot / Paper / Purpur — نسخهٔ **1.16 تا 1.21**
* جاوا: بایت‌کد **Java 8** → روی سرورهای Java 8 تا Java 21 اجرا می‌شود
* وابستگی‌ها: `spigot-api` و `gson` با scope=provided (jar نهایی سبک و بدون shade)

---

## نصب

```bash
# ۱) کپی در پوشهٔ plugins
cp ../dist/BazzarMC-1.0.0.jar /path/to/server/plugins/

# ۲) اجرای سرور (config.yml ساخته می‌شود)
# ۳) ویرایش plugins/BazzarMC/config.yml
# ۴) در کنسول سرور
bmc reload
bmc ping
```

در `config.yml` فقط دو مقدار الزامی است:

```yaml
api:
  base-url: "https://yoursite.com"          # بدون اسلش انتهایی
  token: "توکن از پنل وردپرس"               # پیشخوان ← BazzarMc ← اتصال سرور
```

---

## دستورها

> **نام دستورهای پلاگین کاملاً قابل تغییر است.** در `config.yml` می‌توانید `command.name` (دستور بازیکن) و `command.admin-name` (دستور مدیر) را به هر چیزی که می‌خواهید تغییر دهید و فهرست نام‌های جایگزین (`command.aliases` / `command.admin-aliases`) را هم آزادانه تعیین کنید. دستورهای سفارشی در زمان اجرا ثبت می‌شوند؛ پس از تغییر، سرور را رستارت کنید (یا `bmc reload`). در جدول‌های زیر از نام پیش‌فرض استفاده شده است.


### بازیکنان — `/mclink` (دسترسی: `bazzarmc.use`، پیش‌فرض: همه)

| دستور | کار |
|---|---|
| `/mclink <کد>` | استفاده از کد ۵ دقیقه‌ای پنل کاربری سایت |
| `/mclink phone <شماره>` | ارسال رمز یک‌بارمصرف پیامکی |
| `/mclink phone <شماره> <کد>` | تأیید رمز و اتصال حساب |
| `/mclink mail <ایمیل>` | ارسال رمز از راه ایمیل |
| `/mclink mail <ایمیل> <کد>` | تأیید رمز ایمیلی و اتصال |
| `/mclink status` | وضعیت اتصال + تعداد آیتم‌های در انتظار |
| `/mclink claim` | دریافت دستی آیتم‌های در انتظار |
| `/mclink help` | راهنما |

نام‌های جایگزین: `/storelink`، `/slink`.

### مدیران — `/bmc` (دسترسی: `bazzarmc.admin`، پیش‌فرض: op)

| دستور | کار |
|---|---|
| `/bmc version` | نسخه و وضعیت پیکربندی |
| `/bmc reload` | بارگذاری مجدد `config.yml` و راه‌اندازی دوبارهٔ زمان‌بند |
| `/bmc ping` | تست اتصال به سایت (نسخهٔ افزونه و زمان سرور) |
| `/bmc token` | نمایش توکن و آدرس API فعلی |
| `/bmc deliveries <بازیکن>` | فهرست تحویل‌های در انتظار یک بازیکن |
| `/bmc unlink <بازیکن>` | لغو اتصال یک بازیکن |

---

## تنظیمات

```yaml
api:
  base-url: "https://yoursite.com"
  token: "CHANGE_ME"
  endpoint: "/wp-json/bazzarmc/v1"   # معمولاً تغییر نمی‌کند
  timeout-seconds: 10
  verify-ssl: true                   # false = قبول گواهی خودامضا (هماهنگ با ssl_verify در پنل وردپرس)

store:
  name: "فروشگاه من"                  # متغیر {store}
  url: ""                            # متغیر {store_url} — خالی = همان api.base-url
  ip: ""                             # متغیر {ip} — خالی = تشخیص خودکار از server.properties
  server-name: ""                    # متغیر {server}

command:
  name: "mclink"                     # نام دستور بازیکن (دلخواه مدیر)
  aliases: [bazzar, storelink, slink, link]
  admin-name: "bmc"                  # نام دستور مدیر (دلخواه مدیر)
  admin-aliases: [bazzarmcadmin, bmcadmin]

link:
  broadcast: true                       # اعلام اتصال در چت عمومی
  broadcast-message: "&a» &f{player} &7اکانت خود را به سایت متصل کرد!"
  request-cooldown: 60                  # فاصلهٔ بین دو درخواست کد (ثانیه)

delivery:
  enabled: true
  poll-seconds: 45                      # بررسی دوره‌ای بازیکنان آنلاین
  on-join: true                         # بررسی هنگام ورود
  join-delay-seconds: 5                 # تأخیر پس از ورود تا لود شدن کامل
  unknown-product-commands: []          # دستورهای جایگزین برای محصول تعریف‌نشده
  unknown-product-message: "…"
  mark-unknown-as-delivered: false      # false = در صف می‌ماند تا مدیر رسیدگی کند
```

---

## تعریف محصولات

کلید هر محصول در `config.yml` با «شناسهٔ تحویل» محصول در پنل وردپرس (تب *محصولات*) مطابقت داده می‌شود.
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
  # ۱) فقط دستور کنسول (مثلاً LuckPerms)
  "vip":
    message: "&a&lرتبهٔ VIP &7برای شما فعال شد!"
    commands:
      - "lp user {player} parent add vip"

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

### متغیرهای قابل استفاده در `commands` و `message`

| متغیر | مقدار |
|---|---|
| `{player}` | نام بازیکن |
| `{uuid}` | UUID بدون خط تیره |
| `{item}` | کلید/نام محصول |
| `{amount}` | تعداد سفارش‌داده‌شده |
| `{order}` | شناسهٔ سفارش ووکامرس |
| `{product_id}` | شناسهٔ محصول |
| `{key}` | شناسهٔ تحویل تعیین‌شده در پنل وردپرس |
| `{slug}` | نامک محصول |
| `{sku}` | کد کالای محصول |
| `{store}` | نام فروشگاه (`store.name`) |
| `{store_url}` | آدرس فروشگاه (`store.url`) |
| `{ip}` | آی‌پی/دامنهٔ سرور (`store.ip`) |
| `{server}` | نام سرور (`store.server-name`) |
| `{cmd}` | نام دستور بازیکن با اسلش (مثل `/mclink`) |
| `{admin_cmd}` | نام دستور مدیر با اسلش (مثل `/bmc`) |
| `{plugin_version}` | نسخهٔ پلاگین |

> متغیرهای `{store}`، `{store_url}`، `{ip}`، `{server}`، `{cmd}` و `{admin_cmd}` در **همهٔ پیام‌های `messages`** هم قابل استفاده هستند.

> **نکتهٔ تعداد**: `amount` آیتم = `amount` تنظیمات × `amount` سفارش. یعنی اگر کاربر ۲ عدد «کلید صندوقچه» بخرد و `amount: 3` باشد، ۶ کلید دریافت می‌کند.

### رنگ‌ها

علاوه بر کدهای کلاسیک (`&a`، `&l` و…)، کد رنگ هگز هم پشتیبانی می‌شود (MC 1.16+):

```yaml
message: "&#00ff88متن سبز نئونی!"
```

---

## نحوهٔ کار تحویل

1. زمان‌بند هر `poll-seconds` ثانیه و همچنین هنگام ورود هر بازیکن، فهرست تحویل‌های در انتظار را از `GET /deliveries?player=…` می‌گیرد.
2. برای هر ردیف، بخش متناظر در `products` پیدا می‌شود (شناسهٔ تحویل پنل → نام → شناسهٔ نسخه → شناسهٔ محصول → نامک → SKU).
3. آیتم‌ها به اینونتوری داده می‌شوند (سرریز روی زمین می‌افتد) و دستورهای کنسول اجرا می‌شوند.
4. در پایان `POST /deliveries/mark` فراخوانی می‌شود تا ردیف «تحویل‌شده» شود.
5. اگر اینونتوری **هیچ جای خالی** نداشته باشد، تحویل انجام نمی‌شود و ردیف در صف می‌ماند؛ بازیکن پیام «اینونتوری پر است» می‌گیرد و می‌تواند بعداً `/mclink claim` بزند.
6. اگر محصول در `products` تعریف نشده باشد، `unknown-product-commands` اجرا می‌شود و (بسته به `mark-unknown-as-delivered`) ردیف در صف می‌ماند تا مدیر رسیدگی کند.

همهٔ درخواست‌های شبکه در ترد جداگانه اجرا می‌شوند (`runTaskTimerAsynchronously` + `CompletableFuture`) و هیچ‌کدام سرور را بلوکه نمی‌کنند.

---

## ساخت از سورس

پیش‌نیاز: JDK 17 (یا 11) و Maven 3.6+.

```bash
mvn clean package
# → target/BazzarMC-1.0.0.jar
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
    ├── BazzarMC.java               کلاس اصلی، بارگذاری، پیام‌رسانی
    ├── ApiClient.java                 کلاینت ناهمگام REST (HttpURLConnection + Gson)
    ├── DeliveryService.java           زمان‌بندی، تطبیق محصولات، تحویل آیتم/دستور
    ├── command/LinkCommand.java       /mclink
    ├── command/AdminCommand.java      /bmc
    ├── model/ApiResult.java           پوشش پاسخ JSON
    ├── model/Delivery.java            یک ردیف صف تحویل
    └── util/Msg.java                  رنگ‌ها (& و hex) و جایگزینی متغیرها
```

---

## عیب‌یابی

| نشانه | علت / راه‌حل |
|---|---|
| `پلاگین پیکربندی نشده است` | `api.token` هنوز `CHANGE_ME` است یا `api.base-url` خالی است. |
| `سرور خطا داد (HTTP 403)` | توکن اشتباه است؛ از پنل وردپرس کپی مجدد کنید (`bmc token` برای مقایسه). |
| `HTTP 404` | مسیر REST درست نیست؛ سایت باید پیوندهای یکتا (Permalinks) را روی حالت غیر«ساده» داشته باشد. |
| `خطای ارتباطی: SSLHandshakeException` | گواهی SSL سایت معتبر نیست (یا زنجیرهٔ گواهی ناقص). گواهی را اصلاح کنید. |
| آیتم تحویل می‌شود ولی در صف می‌ماند | فراخوانی `deliveries/mark` ناموفق بوده؛ لاگ کنسول و تب *صف تحویل‌ها* در پنل را ببینید. |
| محصول ناشناخته | کلید بخش `products` با نام محصول ووکامرس (حروف کوچک) یا شناسهٔ محصول یکی نیست. با `/bmc deliveries <بازیکن>` مقدار `item` را ببینید و همان را کلید کنید. |
| پیام‌ها رنگ ندارند | کدها را با `&` بنویسید و `bmc reload` بزنید. |
