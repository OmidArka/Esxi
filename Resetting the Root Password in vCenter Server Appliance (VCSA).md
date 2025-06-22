# 🔧 Resetting the Root Password in vCenter Server Appliance (VCSA)

If you’ve forgotten the **root** password of your vCenter Server Appliance (VCSA), you can reset it through the GRUB bootloader by following these steps.

> ⚠️ **Warning:** This procedure requires console access (via iLO, iDRAC, IPMI, or a hypervisor console like vSphere Web Client) and will briefly interrupt the vCenter service. Use with caution during maintenance windows.

---

## 🖥 Prerequisites

- Console access to the vCenter VM
- vCenter must be powered **on**
- vCenter version: 6.5, 6.7, 7.x, or 8.x (this method applies to all)

---

## 🔄 Steps to Reset the Root Password

### 1. Reboot the vCenter Appliance
From the console (not SSH), restart the VCSA virtual machine.

### 2. Interrupt the GRUB Boot Loader

During boot, when the **GRUB menu** appears:
- Quickly press the **arrow key** to stop the countdown.
- Select the line that starts with `Photon` (default boot option).
- Press **`e`** to edit the boot parameters.

### 3. Modify Kernel Boot Parameters

In the editor:
- Find the line starting with `linux` (or `linuxefi`).
- At the end of the line, add the following:
  ```bash
  rw init=/bin/bash
  ```
- Press **F10** or `Ctrl + X` to boot with this modified entry.

### 4. Remount the Filesystem (if needed)

In the shell prompt that appears:
```bash
mount -o remount,rw /
```

### 5. Reset the Root Password

Run:
```bash
passwd
```

Enter and confirm the new password when prompted.

### 6. Reboot the Appliance

After successfully changing the password, reboot the system:
```bash
exec /sbin/init
```

---

## ✅ After Reset

- You can now log in with the new `root` password via SSH or the VCSA console.
- If the root account is locked or expired, consider enabling it via `chage`:
  ```bash
  chage -l root
  chage -E -1 root
  ```

---

## 🔒 Security Recommendation

After regaining access:
- Store credentials securely (e.g., in a password manager).
- Set a calendar reminder for password expiration if enforced.
- Consider enabling alerting for failed login attempts.

> 📝 This method is supported by VMware for appliance-based vCenter deployments (VCSA) and should be used only when no other recovery method is available.

