[English](README.md)

# Free Cloud Desktop

راه‌اندازی سرورهای موقت Windows و Ubuntu با استفاده از GitHub Actions و دسترسی از طریق Tailscale.

سه Workflow در این پروژه وجود دارد:

| # | محیط | نوع دسترسی | Workflow |
|---|---|---|---|
| ۱ | Windows | دسکتاپ (RDP) | `windows.yml` |
| ۲ | Ubuntu Desktop | دسکتاپ XFCE (RDP) | `ubuntu_desktop.yml` |
| ۳ | Ubuntu | خط فرمان (SSH) | `ubuntu_ssh.yml` |

هر سه Workflow از دو GitHub Secret مشترک استفاده می‌کنند. بسته به Workflow، ابزارهایی مثل FFmpeg، Python، Pillow، ImageMagick و Firefox نیز به‌صورت خودکار نصب می‌شوند.


---

## راه‌اندازی

مراحل اولیه برای هر سه Workflow یکسان است.

### ۱. ساخت GitHub Secretها

به مسیر زیر بروید:

**Repository → Settings → Secrets and variables → Actions → New repository secret**

این دو Secret را بسازید:

| Secret | مقدار |
|---|---|
| `RDP_PASSWORD` | یک پسورد قوی برای کاربر `PersianPl` |
| `TAILSCALE_AUTHKEY` | یک Auth Key از Tailscale |

### ساخت پسورد ۳۲ کاراکتری

**WSL / Linux:**

```bash
tr -dc 'A-Za-z0-9' < /dev/urandom | head -c 32; echo
```

**Windows CMD با PowerShell:**

```cmd
powershell -Command "-join ((48..57)+(65..90)+(97..122) | Get-Random -Count 32 | ForEach-Object {[char]$_})"
```

### ساخت Tailscale Auth Key

از پنل مدیریت Tailscale یک Auth Key بسازید.

این گزینه‌ها را فعال کنید:

- **Reusable** — تا کلید برای چند اجرای Workflow قابل استفاده باشد.
- **Ephemeral** — تا ماشین‌های موقت به‌صورت خودکار حذف شوند.

### ۲. ساختار فایل‌ها

```text
.github/workflows/
├── windows.yml
├── ubuntu_desktop.yml
└── ubuntu_ssh.yml

README.md
README_FA.md
```

### ۳. اجرای Workflow

به مسیر زیر بروید:

**Actions → Workflow موردنظر → Run workflow**

بعد از راه‌اندازی ماشین، آدرس IP مربوط به Tailscale در لاگ‌های Workflow نمایش داده می‌شود.

---

## Windows RDP

یک دسکتاپ گرافیکی کامل Windows.

- **راه‌اندازی:** حدود ۱ دقیقه
- **ابزارها:** FFmpeg + Python + Pillow
- **اتصال:** با `mstsc` در Windows یا Remmina در Linux

| فیلد | مقدار |
|---|---|
| Address | Tailscale IP |
| Username | `PersianPl` |
| Password | مقدار `RDP_PASSWORD` |

---

## Ubuntu Desktop

یک محیط دسکتاپ سبک XFCE به همراه xrdp.

- **راه‌اندازی:** حدود ۳ تا ۵ دقیقه
- **ابزارها:** FFmpeg + Python + Pillow + ImageMagick + Firefox
- **اتصال:** با کلاینت‌های RDP مانند mstsc، Remmina یا Microsoft Remote Desktop

| فیلد | مقدار |
|---|---|
| Address | Tailscale IP |
| Username | `PersianPl` |
| Password | مقدار `RDP_PASSWORD` |

> اگر بعد از اتصال صفحه سیاه دیدید، چند ثانیه صبر کنید یا دوباره متصل شوید تا نشست XFCE کامل بارگذاری شود.

---

## Ubuntu SSH

سبک‌ترین و سریع‌ترین گزینه است و محیط گرافیکی ندارد.

- **راه‌اندازی:** حدود ۱ دقیقه
- **ابزارها:** FFmpeg + Python + Pillow + ImageMagick
- **اتصال:**

```bash
ssh PersianPl@<Tailscale-IP>
```

پسورد:

```text
RDP_PASSWORD
```

این Workflow قابلیت Tailscale SSH با `--ssh` را نیز فعال می‌کند. اگر دستگاه شما در همان شبکه Tailscale باشد، می‌توانید از Tailscale SSH نیز استفاده کنید.

---

## کدام گزینه را استفاده کنم؟

| اگر می‌خواهید... | استفاده کنید |
|---|---|
| اسکریپت اجرا کنید، فایل دانلود کنید یا پردازش انجام دهید | Ubuntu SSH |
| از ابزارهای خط فرمان برای ساخت محتوا استفاده کنید | Ubuntu SSH |
| با محیط گرافیکی Linux کار کنید | Ubuntu Desktop |
| نرم‌افزارهای مخصوص Windows را اجرا کنید | Windows RDP |

---

## نکات مهم

- **ماشین‌ها موقت هستند:** هر اجرا حداکثر ۶ ساعت فعال می‌ماند و بعد از آن ماشین و داده‌هایش حذف می‌شوند. اطلاعات مهم را روی آن نگه ندارید.
- **IP عمومی متغیر است:** IP اینترنتی ممکن است بین اجراهای مختلف تغییر کند؛ بنابراین این روش برای سرویس‌هایی که به IP ثابت نیاز دارند مناسب نیست.
- **مناسب برای:** کارهای کوتاه، پردازش یا دانلود موقت و ساخت محتوا.
- **GitHub Actions:** از منابع GitHub Actions مسئولانه و مطابق شرایط استفاده GitHub استفاده کنید.

---

## رفع اشکال

| مشکل | علت احتمالی | چه چیزی را بررسی کنیم؟ |
|---|---|---|
| `invalid key: unable to validate API key` | کلید Tailscale نامعتبر، منقضی یا ناقص است | یک Auth Key جدید بسازید و مقدار کامل آن را در `TAILSCALE_AUTHKEY` قرار دهید |
| مرحله ساخت کاربر خطا می‌دهد | `RDP_PASSWORD` خالی است یا شرایط پسورد را ندارد | Secret مربوط به `RDP_PASSWORD` را بررسی کنید |
| اتصال RDP/SSH برقرار نمی‌شود | Tailscale روی دستگاه کلاینت فعال نیست | Tailscale را روی کلاینت نصب و وارد حساب شوید |
| صفحه سیاه در Ubuntu Desktop | XFCE هنوز در حال راه‌اندازی است | چند ثانیه صبر کنید یا دوباره متصل شوید |
| `Permission denied` هنگام SSH | پسورد اشتباه است | Secret مربوط به `RDP_PASSWORD` را بررسی کنید |
