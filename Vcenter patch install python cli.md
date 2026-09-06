# آموزش نصب پچ آفلاین vCenter Server از طریق CLI (software-packages)

راهنمای کامل و تست‌شده برای آپدیت/پچ کردن vCenter Server Appliance با استفاده از ISO پچ (نه ISO نصب کامل)، به‌صورت آفلاین از طریق خط فرمان.

> ✅ این روش با موفقیت روی vCenter **8.0.3 (Build 24022515 → 25600417)** تست شده است.

---

## پیش‌نیازها

- [ ] فایل ISO از نوع **Patch/Update** (نه Full Installer) — باید شامل فایل‌هایی مثل `manifest-latest.xml`، `rpm-manifest.json`، `patch-metadata-scripts.zip` باشد
- [ ] دسترسی root به vCenter Appliance از طریق SSH
- [ ] دسترسی به vSphere Client برای اتصال ISO به VM
- [ ] **Snapshot کامل از VM appliance** قبل از شروع (الزامی)
- [ ] فضای آزاد کافی روی دیتاستور (حداقل چند گیگ برای staging)

> ⚠️ **هشدار مهم:** قبل از هر اقدامی حتماً از کل VM appliance اسنپ‌شات بگیرید.

---

## تشخیص نوع صحیح ISO

قبل از شروع، مطمئن شوید ISO از نوع Patch است. بعد از mount کردن:

```bash
ls /mnt/cdrom
```

- ❌ اگر پوشه‌هایی مثل `vcsa-ui-installer`, `vcsa-cli-installer`, `migration-assistant` دیدید → این **ISO نصب کامل (Full Installer)** است و برای این روش کار نمی‌کند.
- ✅ اگر فایل‌هایی مثل `manifest-latest.xml`, `rpm-manifest.json`, `patch-metadata-scripts.zip`, و انبوهی از فایل‌های `.rpm` و `.blob` دیدید → این **ISO پچ صحیح** است.

---

## مرحله ۱: اتصال ISO به VM appliance

از vSphere Client:

1. روی VM appliance vCenter کلیک راست کنید → **Edit Settings**
2. روی **CD/DVD Drive 1** کلیک کنید
3. از dropdown، گزینه **Datastore ISO File** را انتخاب کنید
4. مسیر فایل ISO را Browse و انتخاب کنید
5. تیک **Connected** و **Connect At Power On** را فعال کنید
6. **OK**

---

## مرحله ۲: اتصال SSH و ورود به Shell کامل

```bash
ssh root@<vCenter-IP-or-FQDN>
```

اگر وارد `appliancesh` شدید (پرامپت شبیه `Command>`)، باید shell کامل bash را فعال و وارد شوید:

```bash
shell.set --enabled true
shell
```

پرامپت باید به این شکل تغییر کند:
```
root@localhost [ ~ ]#
```

---

## مرحله ۳: پیدا کردن مسیر اسکریپت software-packages

در برخی نسخه‌های appliance، دستور `software-packages` به‌صورت مستقیم در PATH نیست. مسیر واقعی آن یک اسکریپت پایتون است:

```bash
find / -iname "software-packages*" 2>/dev/null
```

مسیر معمول:
```
/usr/lib/applmgmt/support/scripts/software-packages.py
```

از این به بعد تمام دستورات را با `python3` و مسیر کامل اجرا کنید:

```bash
python3 /usr/lib/applmgmt/support/scripts/software-packages.py <command>
```

---

## مرحله ۴: Mount کردن ISO در سطح سیستم‌عامل

```bash
lsblk
```
دستگاه CD-ROM را پیدا کنید (معمولاً `sr0`).

```bash
mkdir -p /mnt/cdrom
mount /dev/sr0 /mnt/cdrom
ls /mnt/cdrom
```

اگر پیام `WARNING: source write-protected, mounted read-only` دیدید، طبیعی است و مشکلی ایجاد نمی‌کند.

---

## مرحله ۵: Stage کردن پچ

```bash
python3 /usr/lib/applmgmt/support/scripts/software-packages.py stage --iso
```

### نکات مهم حین اجرا:

1. ابتدا وضعیت **Discovering updates** و شناسایی نسخه فعلی/هدف نمایش داده می‌شود.
2. سپس متن کامل **Foundation Agreement (EULA)** نمایش داده می‌شود که چندین صفحه طول دارد.
3. برای رد کردن هر صفحه، کلید **Enter** را بزنید (چندین بار پشت سر هم لازم است).
4. **در انتها**، این سؤال ظاهر می‌شود:
   ```
   Do you accept the terms and conditions?  [yes/no]
   ```
   ⚠️ اینجا حتماً باید کلمه کامل را تایپ کنید، نه فقط Enter بزنید:
   ```
   yes
   ```

بعد از تایید، دانلود و استیج‌کردن بلاب‌ها و RPM‌ها شروع می‌شود و در پایان باید این پیام را ببینید:
```
Staging completed successfully.
Staging process completed successfully
```

---

## مرحله ۶: بررسی وضعیت Staged

```bash
python3 /usr/lib/applmgmt/support/scripts/software-packages.py list --staged
```

خروجی شامل جزئیات پچ staged شده است: نام، نسخه، شماره بیلد، severity، حجم فایل و لیست سرویس‌های تحت تأثیر.

---

## مرحله ۷: نصب پچ

```bash
python3 /usr/lib/applmgmt/support/scripts/software-packages.py install --staged
```

این عملیات معمولاً بین ۳۰ تا ۹۰ دقیقه طول می‌کشد و شامل مراحل زیر است:

```
Running precheck ....
Validating user input ....
Set vmdir maintenance mode ....
Performing LVM based backup/rollback operation ....
Preparing system for update ....
Stopping services ....
Installing packages
Setting up appliance-photon repo and installing RPMS ....
Installing containers ....
Converting data as part of post install ....
Installation completed successfully.
Installation process completed successfully.
```

> ℹ️ اگر پچ از قبل staged بوده باشد، دستور با پیام `update is already staged. Proceeding to install.` مستقیم وارد فاز نصب می‌شود.

---

## مرحله ۸: Reboot appliance

با اینکه ممکن است متادیتای پچ فیلد `rebootrequired: False` داشته باشد، طبق best practice رسمی VMware همیشه بعد از نصب پچ، یک ری‌بوت کامل توصیه می‌شود:

```bash
reboot
```

اتصال SSH قطع می‌شود. چند دقیقه صبر کنید (معمولاً ۵ تا ۱۵ دقیقه) تا appliance کامل بالا بیاید.

---

## مرحله ۹: تایید نهایی

### اتصال مجدد SSH
```bash
ssh root@<vCenter-IP>
```

### بررسی نسخه و بیلد نصب‌شده
```bash
python3 /usr/lib/applmgmt/support/scripts/software-packages.py list --installed
```

### بررسی وضعیت همه سرویس‌ها
```bash
service-control --status --all
```
همه باید در وضعیت `Running` باشند.

### بررسی از طریق VAMI
```
https://<vCenter-IP>:5480 → تب Summary
```
باید Version و Build جدید نمایش داده شود.

### بررسی از طریق vSphere Client
لاگین کنید و مطمئن شوید همه host‌ها و VM‌ها به‌درستی قابل مشاهده و مدیریت هستند.

---

## خلاصه دستورات (Cheat Sheet)

```bash
# ورود به shell کامل
shell.set --enabled true
shell

# mount کردن ISO
mkdir -p /mnt/cdrom
mount /dev/sr0 /mnt/cdrom
ls /mnt/cdrom

# مسیر اسکریپت
SP=/usr/lib/applmgmt/support/scripts/software-packages.py

# استیج کردن (نیاز به تایپ yes در پایان EULA)
python3 $SP stage --iso

# بررسی استیج
python3 $SP list --staged

# نصب
python3 $SP install --staged

# ری‌بوت
reboot

# بعد از بالا اومدن دوباره، تایید نهایی
python3 $SP list --installed
service-control --status --all
```

---

## عیب‌یابی رایج

| مشکل | راه‌حل |
|---|---|
| `command not found` برای `software-packages` | از مسیر کامل با `python3` استفاده کنید: `python3 /usr/lib/applmgmt/support/scripts/software-packages.py` |
| `The CD drive does not have a valid patch ISO` | ISO از نوع Full Installer است، نه Patch. ISO صحیح را از Broadcom Support Portal دانلود کنید |
| بعد از EULA دوباره همان سؤال تکرار می‌شود | فقط Enter خالی زده‌اید؛ باید کلمه `yes` را کامل تایپ کنید |
| `list --staged` می‌گوید `List packages failed` | یعنی staging قبلی ناقص/لغو شده بوده؛ دوباره `stage --iso` را اجرا کنید |
| appliance بعد از reboot بالا نمی‌آید | لاگ را بررسی کنید: `tail -f /var/log/vmware/applmgmt/software-packages.log` |

---

## منابع رسمی

- [VMware vCenter Server Documentation](https://docs.vmware.com/en/VMware-vSphere/index.html)
- [Broadcom Support Portal](https://support.broadcom.com/)
- [vCenter Server Update and Patch Release Notes](https://techdocs.broadcom.com/us/en/vmware-cis/vsphere/vsphere/8-0/release-notes/vcenter-server-update-and-patch-release-notes.html)

---

> این راهنما بر اساس تجربه واقعی نصب پچ VC-8.0U3k (build 24022515 → 25600417) روی vCenter 8.0.3 تهیه شده است.
