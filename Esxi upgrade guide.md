# راهنمای آپگرید ESXi از 7.0.3 به 8.0.3

راهنمای کامل و امن برای آپگرید هاست‌های ESXi بدون از دست رفتن دیتا یا تنظیمات هاردنینگ‌شده.

> ⚠️ مسیر ساپورت‌شده رسمی از **ESXi 7.0 Update 3w** به **8.0 Update 3g** است. بیلد دقیق را از پورتال Broadcom Support متناسب با نسخه فعلی خودتان تایید کنید.

---

## فهرست
1. [پیش‌نیازها](#۱-پیش-نیازها)
2. [بک‌آپ کامل](#۲-بک-آپ-کامل)
3. [ترتیب آپگرید](#۳-ترتیب-آپگرید)
4. [اجرای آپگرید با esxcli](#۴-اجرای-آپگرید-با-esxcli)
5. [بازگردانی کانفیگ و هاردنینگ](#۵-بازگردانی-کانفیگ-و-هاردنینگ)
6. [بررسی نهایی](#۶-بررسی-نهایی)
7. [نکات ریسک‌دار](#۷-نکات-ریسک-دار)

---

## ۱. پیش‌نیازها

- سازگاری سخت‌افزار (NIC، RAID/HBA، فرم‌ور) را در **VMware Compatibility Guide** برای ESXi 8.0 چک کنید.
- اگر VIB یا درایور اختصاصی وندور (HPE/Dell/...) دارید، ایمیج OEM مخصوص 8.0.3 را دانلود کنید، نه ایمیج عمومی.
- **هیچ عملیات live VIB (نصب/حذف/آپدیت زنده) درست قبل از آپگرید انجام ندهید** — طبق مستندات رسمی می‌تواند باعث خرابی ConfigStore شود.

---

## ۲. بک‌آپ کامل

### ۲.۱ بک‌آپ کانفیگ کامل هاست (شامل Advanced Settings)

```bash
vim-cmd hostsvc/firmware/backup_config
```

خروجی یک URL می‌دهد؛ دانلود کنید:

```bash
curl -o configBundle-backup.tgz "http://<IP-هاست>/downloads/xxxxx/configBundle-<hostname>.tgz"
```

### ۲.۲ ثبت Advanced Parameters سفارشی (برای مقایسه دقیق‌تر)

```bash
esxcli system settings advanced list -d > advanced-before.txt
```
فلگ `-d` مقدار default را هم نشان می‌دهد تا بعداً مقادیر دستکاری‌شده را پیدا کنید.

### ۲.۳ بک‌آپ VMها

از snapshot یا ابزار بک‌آپ (Veeam و مشابه) برای VMهای حیاتی استفاده کنید. آپگرید هاست دیتای دیسک VM را دست نمی‌زند، اما همیشه یک نسخه پشتیبان جدا لازم است.

---

## ۳. ترتیب آپگرید

اگر vCenter دارید:

1. اول **vCenter** را به نسخه 8 آپگرید کنید (باید مساوی یا جدیدتر از هاست باشد).
2. سپس هاست‌های ESXi را یکی‌یکی آپگرید کنید.

اگر standalone ESXi دارید، این مرحله را رد کنید.

---

## ۴. اجرای آپگرید با esxcli

### ۴.۱ انتقال فایل depot به دیتاستور

```bash
scp esxi-803-depot.zip root@<IP-هاست>:/vmfs/volumes/<datastore>/
```

نام دیتاستور واقعی را با این دستور پیدا کنید:
```bash
esxcli storage filesystem list
```

### ۴.۲ پیدا کردن نام دقیق پروفایل

```bash
esxcli software sources profile list -d /vmfs/volumes/<datastore>/esxi-803-depot.zip
```

### ۴.۳ Maintenance Mode

```bash
esxcli system maintenanceMode set -e true
```

### ۴.۴ Dry-run (چک بدون اعمال تغییر)

```bash
esxcli software profile update -d /vmfs/volumes/<datastore>/esxi-803-depot.zip -p <profile-name> --dry-run
```

### ۴.۵ اجرای واقعی

```bash
esxcli software profile update -d /vmfs/volumes/<datastore>/esxi-803-depot.zip -p <profile-name>
```

بررسی لاگ در ترمینال دوم:
```bash
tail -f /var/log/esxupdate.log
```

### ۴.۶ ری‌استارت

```bash
reboot
```

---

## ۵. بازگردانی کانفیگ و هاردنینگ

### ۵.۱ اگر همه‌چیز باید عیناً برگردد (کامل)

```bash
scp configBundle-backup.tgz root@<IP-هاست>:/tmp/
esxcli system maintenanceMode set -e true
vim-cmd hostsvc/firmware/restore_config /tmp/configBundle-backup.tgz
```
⚠️ این دستور بعد از اجرا هاست را خودکار ری‌بوت می‌کند و **کل** کانفیگ (شبکه، یوزرها، فایروال، advanced settings) را جایگزین می‌کند.

### ۵.۲ اگر فقط پارامترهای هاردنینگ باید دوباره اعمال شوند

خروجی after را بگیرید و با before مقایسه کنید:

```bash
esxcli system settings advanced list -d > advanced-after.txt
diff advanced-before.txt advanced-after.txt
```

هر پارامتری که ریست شده را دستی برگردانید:

```bash
esxcli system settings advanced set -o /Path/To/Param -i <value>     # عددی
esxcli system settings advanced set -o /Path/To/Param -s "<value>"   # رشته‌ای
```

> اگر هاردنینگ را با **Host Profile** در vCenter اعمال کرده‌اید، بعد از آپگرید باید Profile را دوباره Remediate کنید. همچنین چک‌لیست DISA STIG مخصوص ESXi 8.0 را (که با نسخه 7.0 تفاوت دارد) دوباره بررسی کنید.

---

## ۶. بررسی نهایی

```bash
esxcli system maintenanceMode set -e false
vmware -v
esxcli software profile get
```

---

## ۷. نکات ریسک‌دار

| مورد | توضیح |
|---|---|
| NIC با Media Auto Detect غیرفعال | ممکن است لینک بعد از آپگرید down بماند → `esxcli network nic set -S <speed> -D full -n <nic>` |
| پارامترهای RSS/DRSS سفارشی | در برخی درایورهای شبکه بعد از آپگرید به 8.0 به مقدار پیش‌فرض ریست می‌شوند |
| تغییر نام دستی دیسک رمزنگاری‌شده | خارج از vSphere Client انجام ندهید — می‌تواند vmdk را خراب کند |
| عملیات live VIB قبل از آپگرید | ممکن است ConfigStore را خراب کند و هاست بعد از آپگرید غیرقابل دسترس شود |
| لایسنس | با تغییرات مدل Broadcom، ممکن است نیاز به اکانت Broadcom Support برای دانلود ایمیج و اعمال لایسنس داشته باشید |
