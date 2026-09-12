# مرجع API — BazzarMc

این سند همهٔ مسیرهای ارتباطی افزونه را پوشش می‌دهد:

1. **REST API** (برای پلاگین ماینکرافت و هر کلاینت خارجی)
2. **AJAX داخلی** (برای اسکریپت‌های سمت کاربر سایت)
3. **مسیرهای سازگار با نسخهٔ قدیمی** (`storelinkformc/v1`)
4. **توابع عمومی PHP و شورت‌کدها** (برای قالب و صفحه‌سازها)
5. **ویجت‌های المنتور** (۱۱ ویجت برای طراحی صفحهٔ حساب کاربری)

---

## ۱) احراز هویت REST

همهٔ مسیرها با توکن API محافظت می‌شوند. توکن را از *پیشخوان ← BazzarMc ← اتصال سرور* بگیرید و به یکی از سه روش زیر ارسال کنید:

| روش | نمونه |
|---|---|
| هدر اختصاصی (**پیشنهادی**) | `X-BMC-Token: <token>` |
| هدر Bearer | `Authorization: Bearer <token>` |
| پارامتر | `?token=<token>` |

توکن نامعتبر → `403` با بدنهٔ `{"code":"bmc_invalid_token","message":"توکن نامعتبر است."}`.

**آدرس پایه**

```text
https://yoursite.com/wp-json/bazzarmc/v1
```

همهٔ پاسخ‌ها JSON و با هدرهای `Cache-Control: no-store` ارسال می‌شوند (برای جلوگیری از کش شدن توسط LiteSpeed/WP Rocket/Cloudflare).

---

## ۲) مسیرها

### `GET|POST /ping` — بررسی سلامت

بدون پارامتر.

```json
{
  "success": true,
  "plugin": "BazzarMc",
  "version": "1.3.0",
  "time": "2026-09-12 10:24:33",
  "site": "فروشگاه ماینکرافت من",
  "verify": [ "dashboard", "email" ]
}
```

```bash
curl -H "X-BMC-Token: TOKEN" https://yoursite.com/wp-json/bazzarmc/v1/ping
```

---

### `POST /link/redeem` — استفاده از کد اتصال پنل کاربری

همان مسیری که دستور `/mclink <کد>` در بازی فراخوانی می‌کند.

| پارامتر | نوع | الزامی | توضیح |
|---|---|---|---|
| `code` | string | بله | کد نمایش‌داده‌شده در پنل کاربری |
| `player` | string | بله | نام کاربری ماینکرافت (Java: `^[a-zA-Z0-9_]{3,16}$`) |
| `uuid` | string | – | UUID بدون خط تیره؛ برای اتصال دقیق‌تر ذخیره می‌شود |

**پاسخ موفق (200)**

```json
{
  "success": true,
  "message": "اکانت شما با موفقیت متصل شد.",
  "player": "Notch",
  "user_id": 42,
  "display_name": "علی رضایی"
}
```

**پاسخ خطا (400/403/429)**

```json
{ "success": false, "error": "کد نامعتبر یا منقضی است.", "code": "INVALID" }
```

| `code` | معنی |
|---|---|
| `DISABLED` | روش «کد پنل کاربری» از تنظیمات غیرفعال است (403) |
| `RATE` | محدودیت نرخ — ۱۰ درخواست در هر IP (429) |
| `INVALID` | کد وجود ندارد، منقضی شده یا قبلاً استفاده شده |
| `FORMAT` | نام کاربری ماینکرافت نامعتبر |
| `TAKEN` | این نام کاربری قبلاً به حساب دیگری متصل شده |
| `ERROR` | خطای عمومی |

```bash
curl -X POST https://yoursite.com/wp-json/bazzarmc/v1/link/redeem \
  -H "X-BMC-Token: TOKEN" -H "Content-Type: application/json" \
  -d '{"code":"7KQ2M9","player":"Notch","uuid":"069a79f444e94726a5befca90e38aaf5"}'
```

---

### `POST /link/request` — درخواست رمز یک‌بارمصرف از داخل بازی

| پارامتر | نوع | الزامی | توضیح |
|---|---|---|---|
| `player` | string | بله | نام کاربری ماینکرافت |
| `email` | string | بله | ایمیل حساب کاربری سایت |
| `channel` | string | – | روش تحویل کد: `dashboard` (پیش‌فرض — کد در پاسخ برمی‌گردد و در پنل کاربری و ویجت پیشخوان نمایش داده می‌شود) یا `email` |

**پاسخ موفق**

```json
{
  "success": true,
  "message": "کد تأیید ارسال شد.",
  "channel": "email",
  "expires_in": 120
}
```

خطاها: `DISABLED` (روش غیرفعال)، `RATE` (محدودیت نرخ)، `FORMAT` (ایمیل نامعتبر یا حساب یافت نشد)، `SEND_FAIL` (خطای ارسال ایمیل)، `CHANNEL` (هیچ روش تأییدی در دسترس نیست)، `NO_USER`، `ALREADY_LINKED`، `PLAYER_TAKEN`.

---

### `POST /link/verify` — تأیید رمز و اتصال حساب

| پارامتر | نوع | الزامی | توضیح |
|---|---|---|---|
| `player` | string | بله | نام کاربری ماینکرافت |
| `code` | string | بله | کد دریافتی |
| `email` | string | بله | همان ایمیلی که کد برایش صادر شد |

```json
{ "success": true, "message": "اکانت شما متصل شد!", "player": "Notch" }
```

خطاها: `RATE` (۲۰ تلاش در هر IP)، `FORMAT`، `INVALID` (کد اشتباه/منقضی)، `ATTEMPTS` (بیش از حد مجاز تلاش).

---

### `GET /player` — وضعیت یک بازیکن

| پارامتر | نوع | الزامی |
|---|---|---|
| `player` | string | بله |
| `limit` | int | – (پیش‌فرض ۵۰، حداکثر ۲۰۰) |

**متصل**

```json
{
  "success": true,
  "linked": true,
  "player": "Notch",
  "uuid": "069a79f444e94726a5befca90e38aaf5",
  "user_id": 42,
  "display_name": "علی رضایی",
  "roles": ["customer", "vip"],
  "linked_at": 1757661600,
  "pending": 2
}
```

**متصل‌نشده**

```json
{ "success": true, "linked": false, "player": "Notch" }
```

---

### `GET /deliveries` — صف تحویل‌های در انتظار

| پارامتر | نوع | الزامی | توضیح |
|---|---|---|---|
| `player` | string | بله | نام بازیکن |
| `limit` | int | – | پیش‌فرض ۵۰، حداکثر ۲۰۰ |

```json
{
  "success": true,
  "player": "Notch",
  "count": 2,
  "deliveries": [
    {
      "id": 18,
      "order_id": 1042,
      "order_number": "1042",
      "code": "1042-18",
      "product": "کوین ۱۰۰",
      "rank_level": 0,
      "item": "کوین ۱۰۰",
      "key": "gold_pack_2",
      "slug": "gold_pack_2",
      "sku": "COIN-100",
      "product_id": 355,
      "variation_id": 0,
      "amount": 1,
      "created_at": "2026-09-12 10:15:02"
    }
  ]
}
```

> `item` نام محصول است و `key` همان «شناسهٔ تحویل» است که مدیر در تب **محصولات و نقش‌ها** تعیین می‌کند (نام / شناسهٔ محصول / نامک / SKU / متن دلخواه). پلاگین ماینکرافت برای یافتن محصول در `config.yml` به این ترتیب تلاش می‌کند: `key` → `item` → `variation_id` → `product_id` → `slug` → `sku`.
>
> **فیلدهای نسخهٔ ۱.۲.۰:** `code` = **کد خرید** (پیش‌فرض `شمارهٔ سفارش-شناسهٔ ردیف`) که در منوی دریافت ماینکرافت به‌عنوان نام آیتم کاغذی استفاده می‌شود؛ `product` = نام نمایشی محصول؛ `order_number` = شمارهٔ سفارش ووکامرس؛ `rank_level` = سطح رنک محصول (۰ اگر رنک نباشد).

---

### `POST /deliveries/mark` — ثبت تحویل

| پارامتر | نوع | الزامی |
|---|---|---|
| `id` | int | بله |
| `player` | string | – (برای ثبت در لاگ) |

```json
{ "success": true, "message": "تحویل ثبت شد." }
```

---

### `POST /unlink` — لغو اتصال

| پارامتر | نوع | الزامی |
|---|---|---|
| `player` | string | بله |

```json
{ "success": true, "message": "اتصال حذف شد." }
```

اگر تنظیم «اجازهٔ لغو اتصال» غیرفعال باشد، `403` برمی‌گردد.

---

### `GET /products` — فهرست محصولات سایت (جدید در ۱.۲.۰)

| پارامتر | نوع | الزامی | توضیح |
|---|---|---|---|
| `limit` | int | – | پیش‌فرض ۲۰۰، حداکثر ۵۰۰ |

```json
{
  "success": true,
  "count": 1,
  "products": [
    {
      "id": 355,
      "variation_id": 0,
      "name": "رتبهٔ VIP",
      "slug": "vip-rank",
      "sku": "VIP-30",
      "key": "vip",
      "price": 150000,
      "synced": true,
      "role": "vip",
      "days": 30,
      "rank_level": 1
    }
  ],
  "ranks": [ { "key": "vip", "name": "رتبهٔ VIP", "level": 1, "role": "vip", "days": 30 } ],
  "status": { "enabled": true, "hide_lower": true, "definitions": 3, "manual": 2, "synced_at": 1788000000, "reported_at": 1788003600 },
  "store": { "name": "سرور ماینکرافت", "url": "https://yoursite.com/", "version": "1.3.0" }
}
```

> با `/bmc products` در کنسول سرور بازی، همین فهرست به‌همراه کلید تحویل و سطح رنک هر محصول نمایش داده می‌شود.

---

### `POST /ranks/definitions` — دریافت تعریف رنک‌ها از پلاگین ماینکرافت

بدنهٔ درخواست (از `ranks.list` در `config.yml` پلاگین ساخته می‌شود):

```json
{
  "ranks": [
    { "key": "vip",   "name": "رتبهٔ VIP",  "level": 1, "role": "vip",     "days": 30 },
    { "key": "vip+",  "name": "رتبهٔ VIP+", "level": 2, "role": "vipplus", "days": 30 },
    { "key": "legend","name": "رتبهٔ لجند", "level": 3, "role": "legend",  "days": 0  }
  ]
}
```

```json
{ "success": true, "count": 3, "message": "تعریف ۳ رنک ذخیره شد." }
```

> تعریف‌ها در کلید `mc_ranks` ذخیره می‌شوند و **منبع اصلی سطح رنک هر محصول** هستند؛ «سطح» تعیین‌شدهٔ دستی در تب محصولات بر آن‌ها اولویت دارد.

---

### `POST /ranks/report` — گزارش زمان باقی‌ماندهٔ رنک یک بازیکن

| پارامتر | نوع | الزامی | توضیح |
|---|---|---|---|
| `player` | string | بله | نام بازیکن متصل |
| `uuid` | string | – | UUID بدون خط تیره (برای لاگ) |
| `ranks` | array | بله | فهرست رنک‌های فعال بازیکن |
| `full` | bool | – | پیش‌فرض `true`؛ در این حالت رنک‌هایی که در فهرست نیستند بازپس گرفته می‌شوند |

```json
{
  "player": "Notch",
  "uuid": "069a79f444e94726a5befca90e38aaf5",
  "full": true,
  "ranks": [
    { "key": "vip+", "name": "رتبهٔ VIP+", "level": 2, "role": "vipplus", "days_left": 18, "permanent": false }
  ]
}
```

```json
{ "success": true, "user_id": 42, "updated": 1, "pruned": 0 }
```

> اگر بازیکن متصل پیدا نشود `404` برمی‌گردد. هر رنک در متای `bmc_role_grants` کاربر با `source: minecraft` ثبت/به‌روز می‌شود؛ `days_left: -1` همراه با `permanent: true` یعنی رنک دائمی.

---

### `GET /ranks/status` — وضعیت رنک‌های یک بازیکن

| پارامتر | نوع | الزامی |
|---|---|---|
| `player` | string | – (خالی = فقط وضعیت کلی سامانهٔ رنک) |

```json
{
  "success": true,
  "linked": true,
  "user_id": 42,
  "player": "Notch",
  "top_level": 2,
  "ranks": [
    { "key": "vip+", "name": "رتبهٔ VIP+", "role": "vipplus", "level": 2, "permanent": false,
      "expires_at": "2026-10-01 12:30:00", "days_left": 18, "hours_left": 432 }
  ],
  "status": { "enabled": true, "hide_lower": true, "definitions": 3, "manual": 2, "synced_at": 1788000000, "reported_at": 1788003600 }
}
```

---

## ۳) مسیرهای سازگار با نسخهٔ قدیمی

فقط اگر در تب **اتصال سرور** گزینهٔ «سازگاری با نسخهٔ قدیمی» فعال باشد. آدرس پایه: `…/wp-json/storelinkformc/v1`

| مسیر | معادل جدید | پارامترها |
|---|---|---|
| `POST /request-link` | `/link/request` | `player`, `email` |
| `POST /verify-link` | `/link/verify` | `email`, `code` |
| `GET /pending` | `/deliveries` | `player` |
| `POST /mark-delivered` | `/deliveries/mark` | `id` |

پاسخ‌ها مطابق قالب قدیمی (`{"success":true,"deliveries":[…]}`) هستند تا پلاگین‌های سمت سرور قدیمی بدون تغییر کار کنند.

---

## ۴) AJAX داخلی (سمت کاربر سایت)

آدرس: `POST {site_url}/wp-admin/admin-ajax.php` — همه با پارامتر `security` (nonce) محافظت می‌شوند.

### سامانهٔ تیکت (فرم POST معمولی — بدون AJAX)

> از نسخهٔ ۱.۲.۰، ورود/عضویت با رمز یک‌بارمصرف موبایل از افزونه **حذف شده است** (اکشن‌های `bmc_login_*` دیگر وجود ندارند). رمز یک‌بارمصرف فقط برای **لینک‌کردن اکانت ماینکرافت** استفاده می‌شود (اکشن‌های `bmc_request_link_otp` و `bmc_verify_link_otp` در جدول بعدی).

فرم‌های تیکت به‌صورت POST معمولی به همان آدرس صفحه ارسال می‌شوند و با الگوی «ارسال → تغییر مسیر» (PRG) پاسخ می‌دهند. همه با nonce `bmc_ticket_nonce` محافظت می‌شوند و نیازمند ورود کاربر هستند.

| فیلد `bmc_ticket_action` | سایر فیلدها | توضیح |
|---|---|---|
| `create` | `ticket_subject`, `ticket_body`, `ticket_category`, `ticket_product` | ثبت تیکت جدید |
| `reply` | `ticket_id`, `ticket_body`, `as_staff` (فقط مدیر) | ارسال پاسخ |
| `close` | `ticket_id` | بستن تیکت |
| `reopen` | `ticket_id` | بازکردن دوبارهٔ تیکت |

پس از پردازش، کاربر به آدرس تیکت با پارامتر `bmc_ticket_msg` (موفق) یا `bmc_ticket_error` (خطا) بازمی‌گردد.

پیشخوان مدیریت از `admin-post.php` با `action=bmc_ticket` و nonce یکسان استفاده می‌کند:

| فیلد `ticket_action` | سایر فیلدها | توضیح |
|---|---|---|
| `reply` | `ticket_id`, `ticket_body`, `is_note` | پاسخ پشتیبانی یا یادداشت داخلی |
| `status` | `ticket_id`, `status`, `priority` | تغییر وضعیت/اولویت |
| `delete` | `ticket_id` | حذف تیکت و همهٔ پیام‌ها |

### پنل اتصال کاربر (nonce: `bmc_frontend` — نیازمند ورود کاربر)

| `action` | پارامترها | توضیح |
|---|---|---|
| `bmc_generate_code` | – | ساخت کد اتصال ۵ دقیقه‌ای و بازگرداندن متن آن |
| `bmc_code_state` | – | بازیابی کد فعال (پس از رفرش صفحه) |
| `bmc_revoke_code` | – | باطل‌کردن کد فعال |
| `bmc_unlink` | – | لغو اتصال حساب |
| `bmc_request_link_otp` | `channel`, `player` | درخواست کد اتصال از داخل سایت (ایمیل حساب کاربر یا نمایش در داشبورد) |
| `bmc_verify_link_otp` | `code` | تأیید و اتصال |

### چک‌اوت اختصاصی (nonce: `bmc_public`)

| `action` | پارامترها | توضیح |
|---|---|---|
| `bmc_checkout_summary` | – | بازخوانی جمع سبد/تخفیف/هزینهٔ ارسال |
| `bmc_apply_coupon` | `coupon` | اعمال کد تخفیف |
| `bmc_remove_coupon` | – | حذف کد تخفیف |

ارسال نهایی سفارش با همان مکانیزم استاندارد ووکامرس انجام می‌شود:

```text
POST {site_url}/?wc-ajax=checkout
security = <nonce woocommerce-process_checkout>
```

### AJAX پنل مدیریت (nonce: `bmc_admin` — نیازمند `manage_woocommerce`)

`action=bmc_admin` با پارامتر `do`:

| `do` | کار |
|---|---|
| `regenerate_token` | ساخت توکن API جدید |
| `test_email` | ارسال ایمیل آزمایشی (کد نمونه) |
| `ping_api` | تست اتصال REST |
| `run_cron` | اجرای دستی زمان‌بند نگهداری |
| `clear_logs` | پاک‌کردن لاگ‌ها |
| `rebuild_tables` | بازسازی جدول‌های دیتابیس |
| `create_pages` | ساخت برگه‌های اتصال/سبد خرید/چک‌اوت/تیکت پشتیبانی |
| `purge_cache` | پاک‌سازی کش سایت + افزونه‌های کش + کلادفلر و حذف transientها |
| `cf_cache_rule` | ساخت/به‌روزرسانی قاعدهٔ «کش نشود» در کلادفلر (نیازمند `cf_api_token` و `cf_zone_id`) |
| `run_migration` | مهاجرت از نسخهٔ قدیمی |
| `detect_mobile_keys` | شناسایی کلیدهای متای موبایل |
| `sync_mobiles` | همگام‌سازی شماره‌ها به `bmc_mobile` |
| `mark_delivered` / `reset_pending` / `delete_delivery` | عملیات روی ردیف‌های صف تحویل |

---

## ۵) نمونهٔ کامل با `curl`

```bash
TOKEN="..."
BASE="https://yoursite.com/wp-json/bazzarmc/v1"

# سلامت
curl -sS -H "X-BMC-Token: $TOKEN" "$BASE/ping"

# وضعیت بازیکن
curl -sS -H "X-BMC-Token: $TOKEN" "$BASE/player?player=Notch"

# صف تحویل
curl -sS -H "X-BMC-Token: $TOKEN" "$BASE/deliveries?player=Notch&limit=20"

# ثبت تحویل ردیف ۱۸
curl -sS -X POST -H "X-BMC-Token: $TOKEN" -H "Content-Type: application/json" \
  -d '{"id":18,"player":"Notch"}' "$BASE/deliveries/mark"

# اتصال با کد پنل
curl -sS -X POST -H "X-BMC-Token: $TOKEN" -H "Content-Type: application/json" \
  -d '{"code":"7KQ2M9","player":"Notch"}' "$BASE/link/redeem"

# اتصال با رمز یک‌بارمصرف (ایمیل یا داشبورد) — برای لینک‏کردن اکانت
curl -sS -X POST -H "X-BMC-Token: $TOKEN" -H "Content-Type: application/json" \
  -d '{"player":"Notch","email":"user@example.com","channel":"email"}' "$BASE/link/request"
curl -sS -X POST -H "X-BMC-Token: $TOKEN" -H "Content-Type: application/json" \
  -d '{"player":"Notch","email":"user@example.com","code":"12345"}' "$BASE/link/verify"

# فهرست محصولات سایت
curl -sS -H "X-BMC-Token: $TOKEN" "$BASE/products?limit=100"

# ارسال تعریف رنک‌ها از پلاگین ماینکرافت به سایت
curl -sS -X POST -H "X-BMC-Token: $TOKEN" -H "Content-Type: application/json" \
  -d '{"ranks":[{"key":"vip","name":"رتبهٔ VIP","level":1,"role":"vip","days":30}]}' "$BASE/ranks/definitions"

# گزارش زمان باقی‌ماندهٔ رنک یک بازیکن
curl -sS -X POST -H "X-BMC-Token: $TOKEN" -H "Content-Type: application/json" \
  -d '{"player":"Notch","full":true,"ranks":[{"key":"vip","level":1,"role":"vip","days_left":18,"permanent":false}]}' "$BASE/ranks/report"

# وضعیت رنک‌های یک بازیکن
curl -sS -H "X-BMC-Token: $TOKEN" "$BASE/ranks/status?player=Notch"
```

---

## ۶) نمونهٔ پیاده‌سازی کلاینت (Java)

کلاس `studio.obsifox.bazzarmc.ApiClient` در `mc-plugin/` یک پیاده‌سازی کامل و ناهمگام از همین قرارداد است:

```java
ApiClient api = plugin.getApi();

api.redeemCode(player.getName(), uuid, "7KQ2M9")
   .thenAccept(result -> api.sync(() -> {
       if (result.isSuccess()) {
           player.sendMessage("اتصال برقرار شد!");
       } else {
           player.sendMessage("خطا: " + result.getMessage());
       }
   }));
```

---

## ۷) وضعیت‌های ردیف تحویل

| مقدار `status` | معنی |
|---|---|
| `0` | در انتظار (pending) |
| `1` | تحویل‌شده (delivered) |
| `2` | ناموفق (failed) |
| `3` | منقضی (expired) |

زمان انقضا از تنظیم «انقضای تحویل» در تب **محصولات** می‌آید و زمان‌بند ساعتی (`bmc_hourly_maintenance`) ردیف‌های قدیمی را منقضی می‌کند.

---

## ۸) توابع عمومی PHP و شورت‌کدها (سمت کاربر سایت)

همهٔ این توابع رشتهٔ HTML برمی‌گردانند (چاپ نمی‌کنند) و دارایی‌های لازم را خودکار بارگذاری می‌کنند:

| تابع | خروجی | شورت‌کد معادل |
|---|---|---|
| `BMC_Public::shortcode_account( $atts )` | پنل اتصال اکانت ماینکرافت (کد ۵ دقیقه‌ای + تأیید) | `[bazzarmc]` / `[bazzarmc_account]` / `[bazzarmc_link]` |
| `BMC_Public::shortcode_status()` | نشان وضعیت اتصال | `[bazzarmc_status]` |
| `BMC_Public::shortcode_cart()` | سبد خرید اختصاصی + پرداخت | `[bazzarmc_cart]` |
| `BMC_Public::shortcode_checkout()` | چک‌اوت اختصاصی یا درگاه اتصال | `[bazzarmc_checkout]` |
| `BMC_Public::render_account_card( $args )` | کارت حساب کاربری (بدون قاب بیرونی) | `[bazzarmc_card title="…" links="orders,cart" layout="grid"]` |
| `BMC_Public::render_account_nav( $args )` | منوی بخش‌های حساب کاربری ووکامرس | `[bazzarmc_nav layout="pills"]` |
| `BMC_Public::render_account_content( $endpoint, $args )` | محتوای یک اندپوینت ووکامرس | — |
| `BMC_Public::render_ranks( $args )` | فهرست رنک‌های فعال کاربر | `[bazzarmc_ranks title="…"]` |
| `BMC_Public::render_deliveries( $args )` | فهرست تحویل‌های کاربر | `[bazzarmc_deliveries status="pending" limit="10" layout="table"]` |
| `BMC_Tickets::shortcode_tickets()` | فهرست تیکت‌ها + فرم ثبت | `[bazzarmc_tickets]` |
| `BMC_Tickets::shortcode_ticket( $atts )` | گفت‌وگوی یک تیکت | `[bazzarmc_ticket id="12"]` |
| `BMC_Deliveries::for_user( $user_id, $args )` | آرایهٔ ردیف‌های تحویل یک کاربر (بر اساس `user_id` یا بازیکن متصل) | — |
| `BMC_Deliveries::counts_for_user( $user_id )` | شمار وضعیت‌های تحویل یک کاربر | — |

**پارامترهای `render_account_card()`** (همه اختیاری): `user_id`، `title`، `title_icon`، `layout` (`grid`/`stack`)، `columns` (۱ تا ۴)، `kicker`، `kicker_text`، `email`، `mobile`، `orders`، `roles`، `mc_box`، `ranks_box`، `links`، `link_items` (آرایه‌ای از `orders`، `addresses`، `details`، `cart`، `minecraft`، `tickets`، `logout`)، `avatar_size`، `unlinked_text`، `no_ranks_text`، `manage_label`، `connect_label`.

**پارامترهای `render_account_nav()`**: `layout` (`vertical`/`horizontal`/`pills`)، `icons`، `counts`، `active_only`، `title`، `title_icon`، `items` (فهرست اندپوینت‌های مجاز — خالی = همه)، `extra` (آرایهٔ موارد دلخواه با کلیدهای `label`، `url`، `icon`). آیکون هر اندپوینت با فیلتر `bmc_account_nav_icons` قابل تغییر است.

**پارامترهای `render_account_content()`**: آرگومان اول نام اندپوینت است (`current`، `dashboard`، `orders`، `downloads`، `edit-address`، `payment-methods`، `edit-account`، `minecraft`، `bmc-tickets`، `bmc-ticket`) و آرگومان دوم آرایهٔ `hide_notices`، `address` (`billing`/`shipping`) و `order_id`.

---

## ۹) ویجت‌های المنتور

اگر المنتور (نسخهٔ ۳.۱ یا جدیدتر) فعال باشد، ۱۱ ویجت در دستهٔ **«BazzarMc — فروشگاه ماینکرافت»** ثبت می‌شود:

| نام ویجت (name) | عنوان در پنل |
|---|---|
| `bmc-account-card` | کارت حساب کاربری |
| `bmc-link-box` | اتصال اکانت ماینکرافت |
| `bmc-status` | وضعیت اتصال |
| `bmc-account-nav` | منوی حساب کاربری |
| `bmc-account-content` | محتوای حساب کاربری |
| `bmc-cart` | سبد خرید اختصاصی |
| `bmc-checkout` | چک‌اوت اختصاصی |
| `bmc-tickets` | تیکت‌های پشتیبانی |
| `bmc-ticket` | گفت‌وگوی تیکت |
| `bmc-deliveries` | تحویل‌های من |
| `bmc-ranks` | رنک‌های کاربر |

* بارگذاری: `includes/integrations/class-bmc-elementor.php` (کلاس `BMC_Elementor`) + `includes/integrations/elementor/`
* کلید تنظیمات: `elementor_enabled` (پیش‌فرض روشن) — بخش «یکپارچه‌سازی المنتور» در تب **ابزارها**
* دارایی‌ها: `bmc-public`، `bmc-elementor` (و `bmc-vazir` اگر CDN فونت روشن باشد) به‌صورت `get_style_depends()` و `bmc-icons`/`bmc-link`/`bmc-checkout` به‌صورت `get_script_depends()`
* **بدون المنتور**: هیچ کلاسی بارگذاری نمی‌شود (`did_action('elementor/loaded')` بررسی می‌شود) و افزونه هیچ خطا یا هشداری تولید نمی‌کند.
