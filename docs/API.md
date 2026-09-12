# مرجع API — BazzarMc

این سند همهٔ مسیرهای ارتباطی افزونه را پوشش می‌دهد:

1. **REST API** (برای پلاگین ماینکرافت و هر کلاینت خارجی)
2. **AJAX داخلی** (برای اسکریپت‌های سمت کاربر سایت)
3. **مسیرهای سازگار با نسخهٔ قدیمی** (`storelinkformc/v1`)

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
  "version": "1.0.0",
  "time": "2026-09-12 10:24:33",
  "site": "فروشگاه ماینکرافت من",
  "sms": true
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
| `mobile` | string | یکی از دو | شمارهٔ موبایل (مثل `09123456789`) |
| `email` | string | یکی از دو | ایمیل کاربر |
| `channel` | string | – | روش تحویل کد: `sms` (پیش‌فرض)، `email` یا `dashboard` — روش «داشبورد» کد را در پاسخ برمی‌گرداند و هزینهٔ پیامکی ندارد |

**پاسخ موفق**

```json
{
  "success": true,
  "message": "کد تأیید ارسال شد.",
  "channel": "mobile",
  "expires_in": 120
}
```

خطاها: `DISABLED` (روش غیرفعال)، `RATE` (محدودیت نرخ)، `FORMAT` (شماره/ایمیل نامعتبر یا حساب یافت نشد)، `SMS` (خطای سرویس پیامک).

---

### `POST /link/verify` — تأیید رمز و اتصال حساب

| پارامتر | نوع | الزامی | توضیح |
|---|---|---|---|
| `player` | string | بله | نام کاربری ماینکرافت |
| `code` | string | بله | کد دریافتی |
| `mobile` | string | یکی از دو | همان شماره‌ای که کد برایش ارسال شد |
| `email` | string | یکی از دو | همان ایمیلی که کد برایش ارسال شد |
| `channel` | string | – | همان روشی که کد با آن صادر شد (`sms`/`email`/`dashboard`) |

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

> `item` نام محصول است و `key` همان «شناسهٔ تحویل» است که مدیر در تب **محصولات** تعیین می‌کند (نام / شناسهٔ محصول / نامک / SKU / متن دلخواه). پلاگین ماینکرافت برای یافتن محصول در `config.yml` به این ترتیب تلاش می‌کند: `key` → `item` → `variation_id` → `product_id` → `slug` → `sku`.

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

### ورود با رمز یک‌بارمصرف (nonce: `bmc_public`)

| `action` | پارامترها | توضیح |
|---|---|---|
| `bmc_login_request` | `mobile` | ارسال کد ورود |
| `bmc_login_verify` | `mobile`, `code` | تأیید و ورود (کوکی نشست ست می‌شود) |
| `bmc_login_resend` | `mobile`, `resend=1` | ارسال مجدد |

```json
{ "success": true, "data": { "message": "کد ارسال شد.", "expires_in": 120, "resend_in": 90 } }
```

### پنل اتصال کاربر (nonce: `bmc_frontend` — نیازمند ورود کاربر)

| `action` | پارامترها | توضیح |
|---|---|---|
| `bmc_generate_code` | – | ساخت کد اتصال ۵ دقیقه‌ای و بازگرداندن متن آن |
| `bmc_code_state` | – | بازیابی کد فعال (پس از رفرش صفحه) |
| `bmc_revoke_code` | – | باطل‌کردن کد فعال |
| `bmc_unlink` | – | لغو اتصال حساب |
| `bmc_request_link_otp` | `identifier_type`, `mobile` | درخواست کد اتصال از داخل سایت |
| `bmc_verify_link_otp` | `identifier_type`, `mobile`, `code` | تأیید و اتصال |

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
| `test_sms` | ارسال پیامک آزمایشی |
| `ping_api` | تست اتصال REST |
| `run_cron` | اجرای دستی زمان‌بند نگهداری |
| `clear_logs` | پاک‌کردن لاگ‌ها |
| `rebuild_tables` | بازسازی جدول‌های دیتابیس |
| `create_pages` | ساخت برگه‌های اتصال/ورود/سبد خرید/چک‌اوت |
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

# اتصال با رمز پیامکی
curl -sS -X POST -H "X-BMC-Token: $TOKEN" -H "Content-Type: application/json" \
  -d '{"player":"Notch","mobile":"09123456789"}' "$BASE/link/request"
curl -sS -X POST -H "X-BMC-Token: $TOKEN" -H "Content-Type: application/json" \
  -d '{"player":"Notch","mobile":"09123456789","code":"12345"}' "$BASE/link/verify"
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
