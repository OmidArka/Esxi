# راهنمای آپدیت آفلاین vCenter Server (8.0.3 Build 20411 → 26411)

راهنمای کامل آپدیت آفلاین vCenter Server Appliance با استفاده از فایل ISO، بدون نیاز به دسترسی اینترنت.

## پیش‌نیازها

- [ ] فایل ISO مربوط به بیلد هدف (26411) — **نه** ISO نصب اولیه
- [ ] دسترسی root به vCenter Appliance
- [ ] دسترسی به vSphere Client برای مدیریت VM
- [ ] حداقل ۵۰-۱۰۰ گیگابایت فضای آزاد روی دیتاستور appliance
- [ ] **Snapshot یا Backup کامل از VM** قبل از هرگونه اقدام

> ⚠️ **هشدار:** حتماً قبل از شروع از کل VM اپلاینس Snapshot بگیرید تا در صورت بروز مشکل امکان بازگشت وجود داشته باشد.

---

## مرحله ۰: تهیه Snapshot

از vSphere Client:

```
VM → کلیک راست → Snapshots → Take Snapshot
```

Snapshot را حداقل تا چند روز بعد از آپدیت موفق نگه دارید.

---

## روش ۱: آپدیت از طریق CLI (SSH)

### ۱. اتصال ISO به VM

از vSphere Client:

1. روی VM appliance vCenter کلیک راست کنید → **Edit Settings**
2. **CD/DVD Drive 1** → از dropdown گزینه **Datastore ISO File** را انتخاب کنید
3. مسیر ISO را Browse و انتخاب کنید
4. تیک **Connected** و **Connect At Power On** را فعال کنید
5. **OK**

### ۲. اتصال SSH

```bash
ssh root@<vCenter-IP-or-FQDN>
```

اگر SSH غیرفعال است، از VAMI (`https://<IP>:5480`) → **Access** → **Enable SSH Login**

### ۳. ورود به Shell کامل

```bash
shell
```

اگر غیرفعال بود، از `appliancesh`:

```bash
shell.set --enabled true
```

### ۴. بررسی نسخه فعلی

```bash
cat /etc/applmgmt/appliance/software_update_state.conf
vpxd -v
```

### ۵. اسکن آپدیت از روی CD-ROM

```bash
software-packages list --iso
```

باید بیلد `26411` در خروجی دیده شود.

### ۶. Stage کردن آپدیت

```bash
software-packages stage --iso
```

بررسی وضعیت:

```bash
software-packages list --staged
```

### ۷. نصب آپدیت

```bash
software-packages install --staged
```

> ⚠️ این دستور appliance را چند بار reboot می‌کند و اتصال SSH قطع می‌شود — طبیعی است.

### ۸. بررسی پیشرفت بعد از reconnect

```bash
ssh root@<vCenter-IP>
```

زمان کامل شدن معمولاً بین ۳۰ تا ۹۰ دقیقه است.

### ۹. تایید نسخه جدید

```bash
software-packages list --installed
```

### دیدن لاگ در صورت بروز خطا

```bash
tail -f /var/log/vmware/applmgmt/software-packages.log
```

---

## روش ۲: آپدیت از طریق VAMI (رابط گرافیکی)

| مرحله | عملیات |
|---|---|
| ۱ | ورود به `https://<vCenter-IP-or-FQDN>:5480` با یوزر `root` |
| ۲ | رفتن به تب **Update** |
| ۳ | کلیک روی آیکون سه‌نقطه (⋮) → **CHECK CD ROM** |
| ۴ | مشاهده بیلد 26411 در لیست و کلیک روی آن برای مشاهده جزئیات |
| ۵ | کلیک روی **STAGE AND INSTALL** |
| ۶ | تیک پذیرش EULA → **NEXT** |
| ۷ | بررسی نتیجه Prerequisite Check → **FINISH** |
| ۸ | انتظار برای نصب و ری‌استارت خودکار appliance |
| ۹ | ورود مجدد به VAMI و بررسی **Summary** برای تایید Version: `8.0.3` و Build: `26411` |

---

## بررسی نهایی بعد از آپدیت

- [ ] تب **Summary** در VAMI → Version و Build صحیح است
- [ ] همه سرویس‌ها در vSphere Client به وضعیت **Running** برگشته‌اند
- [ ] لاگین به vSphere Client با موفقیت انجام می‌شود
- [ ] در صورت وجود Enhanced Linked Mode، سایر vCenter ها هم sync هستند
- [ ] Snapshot را حداقل چند روز نگه دارید قبل از حذف

---

## عیب‌یابی رایج

| مشکل | راه‌حل |
|---|---|
| `software-packages list --iso` چیزی نشان نمی‌دهد | بررسی کنید ISO واقعاً به CD-ROM وصل (Connected) است، نه فقط انتخاب‌شده |
| appliance بعد از reboot بالا نمی‌آید | چک کردن `software-packages.log` برای یافتن خطای دقیق |
| فضای دیسک ناکافی | آزاد کردن فضا در دیتاستور appliance قبل از staging |
| Prerequisite Check Fail می‌شود | بررسی پیام دقیق خطا و رفع مشکل قبل از ادامه |

---

## منابع رسمی

- [VMware vCenter Server Documentation](https://docs.vmware.com/en/VMware-vSphere/index.html)
- [Broadcom Support Portal](https://support.broadcom.com/)

---

> این راهنما بر اساس تجربه آپدیت آفلاین از بیلد `20411` به `26411` روی vCenter 8.0.3 تهیه شده است.
