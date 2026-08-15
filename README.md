# Windows Cloud RDP (via GitHub Actions + Tailscale)

راه‌اندازی یک محیط دسکتاپ ویندوز موقت با استفاده از GitHub Actions، امن‌شده پشت شبکه‌ی خصوصی Tailscale.

---

## ✨ امکانات

- 🖥️ دسکتاپ Windows Server با ۱۶ گیگ رم و نت پرسرعت
- 🔒 اتصال امن پشت Tailscale (بدون باز کردن پورت عمومی)
- 🔑 پسورد ادمین از GitHub Secrets خوانده می‌شود (داخل کد ذخیره نمی‌شود)
- 🎨 نصب خودکار ابزارهای ساخت محتوا: FFmpeg، Python، Pillow
- ⏱️ هر اجرا تا ۶ ساعت فعال می‌ماند (سقف GitHub Actions)

---

## ⚙️ راه‌اندازی

### ۱. ساخت Secretها در گیت‌هاب
مسیر: **repo → Settings → Secrets and variables → Actions → New repository secret**

| نام Secret | مقدار |
|---|---|
| `RDP_PASSWORD` | یک پسورد قوی (حداقل ۱۲ کاراکتر، شامل حرف بزرگ/کوچک/عدد) |
| `TAILSCALE_AUTHKEY` | کلید auth از پنل Tailscale |

**ساخت پسورد ۳۲ کاراکتری (در WSL/Linux):**
```bash
tr -dc 'A-Za-z0-9' < /dev/urandom | head -c 32; echo
```

**ساخت پسورد ۳۲ کاراکتری (در CMD ویندوز — با PowerShell):**
```cmd
powershell -Command "-join ((48..57)+(65..90)+(97..122) | Get-Random -Count 32 | ForEach-Object {[char]$_})"
```
> نکته: پسورد باید پیچیدگی ویندوز را داشته باشد (حرف بزرگ + کوچک + عدد). اگر خطای پیچیدگی گرفتید، دوباره اجرا کنید یا یک علامت مثل `@` به ابتدای آن اضافه کنید.

**ساخت Tailscale Auth Key:**
https://login.tailscale.com/admin/settings/keys → Generate auth key
- ✅ **Reusable** را فعال کنید (وگرنه بعد از یک بار استفاده منقضی می‌شود)
- ✅ **Ephemeral** را فعال کنید (سرورهای موقت خودکار پاک شوند)

### ۲. اجرا
مسیر: **Actions → Windows Cloud RDP → Run workflow**

پس از حدود ۱ دقیقه، در لاگِ مرحله‌ی Tailscale، آدرس IP سرور نمایش داده می‌شود.

### ۳. اتصال
با هر کلاینت RDP (مثل `mstsc` در ویندوز یا Remmina در لینوکس) به Tailscale IP وصل شوید:

| فیلد | مقدار |
|---|---|
| Address | Tailscale IP نمایش‌داده‌شده در لاگ |
| Username | `PersianPl` |
| Password | همان مقدار Secret `RDP_PASSWORD` |

---

## 📁 ساختار

```
.github/workflows/
  └── rdp.yml        # فایل workflow اصلی
README.md
```

---

## ⚠️ نکات مهم

- **موقت است:** هر اجرا حداکثر ۶ ساعت زنده می‌ماند، سپس ماشین و داده‌هایش پاک می‌شوند. چیز مهمی روی آن ذخیره نکنید (همیشه بک‌آپ بگیرید).
- **IP خروجی ثابت نیست:** با هر اجرا IP اینترنتی سرور عوض می‌شود؛ برای سرویس‌های حساس به IP (مثل لاگین حساب‌ها) مناسب نیست.
- **مناسب برای:** کارهای کوتاه، پردازش/دانلود سنگین موقت، و ساخت محتوا.
- **رعایت قوانین:** استفاده‌ی مسئولانه از منابع GitHub Actions طبق شرایط سرویس گیت‌هاب.

---

## 🛠️ رفع اشکال

| خطا | علت | راه‌حل |
|---|---|---|
| `invalid key: unable to validate API key` | کلید Tailscale منقضی/نامعتبر | کلید جدید Reusable بسازید و در Secret به‌روزرسانی کنید |
| مرحله‌ی ساخت اکانت خطا می‌دهد | `RDP_PASSWORD` خالی یا ضعیف | Secret را با پسورد قوی بسازید |
| اتصال RDP برقرار نمی‌شود | Tailscale روی دستگاه شما فعال نیست | Tailscale را روی کلاینت خود نصب و لاگین کنید |
