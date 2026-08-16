# ساخت کاربر محلی (Local User) روی vCenter Server Appliance (VCSA)

## هدف
ساخت یک کاربر لوکال روی سیستم‌عامل appliance (Photon OS) که مثل `root` بدون نیاز به `@vsphere.local` وارد بشه — برخلاف کاربرهای SSO که همیشه باید با فرمت `username@domain` لاگین کنن.

> ⚠️ **هشدار:** این روش رسماً توسط VMware/Broadcom توصیه نمی‌شه. کاربران لوکال appliance از چرخه‌ی مدیریت پسورد و پالیسی‌های SSO خارج هستن و ممکنه در آپدیت/پچ VCSA دچار مشکل بشن یا حذف بشن. فقط برای مصارف خاص (مثل دسترسی اضطراری یا تست) استفاده کنید.

## پیش‌نیاز
دسترسی SSH با کاربر `root` به آدرس appliance.

## مراحل

### ۱. اتصال SSH به appliance
```bash
ssh root@<vcsa-ip>
```
با ورود، وارد `appliancesh` (شل محدود مدیریتی) می‌شید، نه bash واقعی.

### ۲. فعال‌سازی bash shell (در صورت غیرفعال بودن)
```bash
shell.set --enabled true
shell
```
بعد از دستور `shell`، باید پرامپت به شکل `root@vcsa [ ~ ]#` تغییر کنه — یعنی الان وارد bash واقعی شدید.

### ۳. ساخت کاربر جدید
```bash
useradd -m -s /bin/bash arootvc
passwd aroot
```

### ۴. اضافه کردن به گروه با دسترسی مدیریتی
```bash
usermod -aG wheel aroot
```

### ۵. تست ورود
```bash
ssh aroot@<vcsa-ip>
```

## عیب‌یابی خطاهای رایج

### خطای `useradd: Permission denied` یا `cannot lock /etc/passwd`
سه علت رایج رو چک کنید:

**الف) در appliancesh مونده‌اید نه bash واقعی**
```bash
whoami
id
```
اگه هنوز تو appliancesh هستید، `shell` رو بزنید تا وارد bash بشید.

**ب) فایل‌سیستم root به‌صورت read-only مانت شده**
```bash
mount | grep " / "
df -h /
```
اگه خروجی `ro` نشون داد یا دیسک پر بود (نزدیک ۱۰۰٪)، این علت مشکل عدم امکان ساخت یوزره.

**ج) لاک فایل باقی‌مونده (stale lock)**
```bash
ls -la /etc/.pwd.lock
rm -f /etc/.pwd.lock   # فقط اگه مطمئنید پروسه‌ی دیگه‌ای درحال استفاده نیست
```

## تفاوت کاربر لوکال appliance و کاربر SSO

| | کاربر SSO (`arootvc@vsphere.local`) | کاربر لوکال appliance (`root`, `arootvc`) |
|---|---|---|
| منبع | دیتابیس SSO | فایل `/etc/passwd` خود appliance |
| فرمت لاگین | همیشه با `@domain` | فقط با username ساده |
| دسترسی VAMI | با عضویت در گروه `Administrators` یا `SystemConfiguration.*` | معمولاً از طریق shell/wheel |
| مدیریت پسورد | از طریق SSO policy | مستقل، دستی |
