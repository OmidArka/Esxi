# رفع قفل اکانت Root در vCenter Server Appliance (VCSA)

راهنمای کامل برای باز کردن قفل اکانت root، فعال‌سازی Bash Shell، و ریست پسورد در VCSA (نسخه‌های 7.x و 8.x).

---

## فهرست
1. [تشخیص مشکل](#۱-تشخیص-مشکل)
2. [روش ۱: باز کردن قفل از طریق SSO Admin (بدون ری‌بوت)](#۲-روش-۱-باز-کردن-قفل-از-طریق-sso-admin-بدون-ری-بوت)
3. [فعال‌سازی Bash Shell (اگه Disable بود)](#۳-فعال-سازی-bash-shell-اگه-disable-بود)
4. [ریست پسورد Root](#۴-ریست-پسورد-root)
5. [روش ۲: دسترسی اضطراری از طریق کنسول VM و GRUB](#۵-روش-۲-دسترسی-اضطراری-از-طریق-کنسول-vm-و-grub)
6. [پیشگیری از تکرار مشکل](#۶-پیشگیری-از-تکرار-مشکل)

---

## ۱. تشخیص مشکل

علائم رایج:
- خطای `Account locked` هنگام لاگین به VAMI (`https://<vCenter-IP>:5480`) یا SSH.
- خطای `shell is disabled` هنگام تلاش برای اجرای دستور `shell` در appliance shell.

بررسی وضعیت قفل (در صورت دسترسی به shell):
```bash
pam_tally2 --user=root
# یا در نسخه‌های جدیدتر (Photon OS 4+، vCenter 8.0 U2 به بعد):
/usr/sbin/faillock --user root
```

---

## ۲. روش ۱: باز کردن قفل از طریق SSO Admin (بدون ری‌بوت)

این روش فقط در صورتی کار می‌کند که اکانت `administrator@vsphere.local` (یا معادل SSO Domain شما) هنوز فعال باشد.

### اتصال SSH
```bash
ssh administrator@vsphere.local@<IP-vCenter>
```

### باز کردن قفل root

برای **vCenter 7.x یا 8.0 اولیه**:
```bash
sudo pam_tally2 --user=root --reset
```

برای **vCenter 8.0 U2 به بعد**:
```bash
sudo faillock --user root --reset
```

> اگر مشکل فقط قفل‌شدگی بوده (نه فراموشی پسورد)، همین مرحله کافی است و می‌توانید با پسورد قبلی وارد شوید.

---

## ۳. فعال‌سازی Bash Shell (اگه Disable بود)

اگر پیام `shell is disabled` دریافت کردید:

### گزینه الف — از طریق appliance shell
داخل محیط محدود appliance (همان `Command>`):
```
shell.set --enabled True
shell
```

### گزینه ب — از طریق وب VAMI
1. وارد شوید: `https://<IP-vCenter>:5480`
2. با `administrator@vsphere.local` لاگین کنید.
3. تب **Access** → گزینه **Bash Shell** را Enable کنید.

---

## ۴. ریست پسورد Root

اگر علاوه بر قفل بودن، پسورد را هم فراموش کرده‌اید:

```bash
sudo /usr/lib/vmware-vmafd/bin/dir-cli password reset --account root
```

یا از طریق appliance shell:
```
localaccounts.user.password.update --username root --password <new-password>
```

---

## ۵. روش ۲: دسترسی اضطراری از طریق کنسول VM و GRUB

استفاده کنید فقط اگر **هیچ‌کدام** از اکانت‌های SSO admin و root در دسترس نیستند.

⚠️ **پیش از انجام: از VM یک Snapshot بگیرید.**

1. از vSphere Client یا ESXi Host Client، روی VM مربوط به vCenter کلیک راست → **Open Remote Console**.
2. VM را Restart کنید (Guest OS → Restart).
3. در صفحه بوت GRUB، کلید `e` را بزنید تا وارد Edit Mode شوید.
4. خط شروع‌شده با `linux` را پیدا کرده و به انتهای آن اضافه کنید:
   ```
   rw init=/bin/bash
   ```
5. کلید `F10` را بزنید تا با این پارامتر بوت شود.
6. فایل‌سیستم روت را با دسترسی نوشتن mount کنید:
   ```bash
   mount -o remount,rw /
   ```
7. قفل را باز کنید:
   ```bash
   pam_tally2 --user=root --reset
   # یا
   /usr/sbin/faillock --user root --reset
   ```
8. در صورت نیاز پسورد جدید تنظیم کنید:
   ```bash
   passwd root
   ```
9. VM را Restart کنید تا به‌صورت عادی بوت شود.

---

## ۶. پیشگیری از تکرار مشکل

### جلوگیری از انقضای پسورد root
```bash
chage -I -1 -m 0 -M 99999 -E -1 root
```
یا تنظیم انقضا به تعداد روز دلخواه:
```bash
chage -M 9999 root
```

### بررسی وضعیت فعلی انقضای پسورد
```bash
chage -l root
```

### تنظیم از طریق VAMI (جایگزین CLI)
`https://<IP-vCenter>:5480` → **Administration** → **Password Expiration** → تنظیم `Password expires` روی `No`.

---

## خلاصه دستورات پرکاربرد

| هدف | دستور |
|---|---|
| چک وضعیت قفل (قدیمی) | `pam_tally2 --user=root` |
| چک وضعیت قفل (جدید) | `faillock --user root` |
| باز کردن قفل (قدیمی) | `pam_tally2 --user=root --reset` |
| باز کردن قفل (جدید) | `faillock --user root --reset` |
| فعال‌سازی Bash Shell | `shell.set --enabled True` |
| ریست پسورد root | `dir-cli password reset --account root` |
| جلوگیری از انقضای پسورد | `chage -I -1 -m 0 -M 99999 -E -1 root` |
