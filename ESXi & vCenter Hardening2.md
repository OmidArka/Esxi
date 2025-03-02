# **ESXi & vCenter Hardening Guide**

## **1. Hardware & Firmware Updates**

### **1.1. Update Hardware Firmware**
- Ensure all server hardware components (BIOS, RAID controller, NICs, etc.) are updated to the latest version provided by the vendor.

### **1.2. Update ESXi to Latest Patch**
#### **Check Current Version:**
```bash
vmware -vl
```
#### **List Installed Security Updates:**
```bash
esxcli software vib list | grep -i security
```
#### **Install Patch Manually:**
1. Upload the patch to the datastore.
2. Put ESXi in maintenance mode:
   ```bash
   esxcli system maintenanceMode set --enable true
   ```
3. Install the patch:
   ```bash
   esxcli software vib update -d /vmfs/volumes/datastore1/patch.zip
   ```
4. Reboot ESXi:
   ```bash
   reboot
   ```

### **1.3. Update vCenter to Latest Version**
- Update via VAMI: `https://vcenter-ip:5480`
- Check and install updates under **Update** section.

### **1.4. Update VMware Tools on VMs**
- Update VMware Tools for all virtual machines to ensure compatibility and security.

---

## **2. User & Authentication Hardening**

### **2.1. Create a Non-Root Admin User**
```bash
esxcli system account add --id=adminuser --password=StrongPass! --description="Admin User"
```

### **2.2. Enforce Strong Password Policy**
- Set password complexity requirements in `/etc/pam.d/passwd`.

### **2.3. Configure Password History & Failed Login Attempts**
```bash
esxcli system settings advanced set -o /Security/PasswordHistory -i 5
esxcli system settings advanced set -o /Security/FailedLoginAttempts -i 5
esxcli system settings advanced set -o /Security/AccountLockoutDuration -i 900
```

### **2.4. Configure Session Timeouts**
```bash
esxcli system settings advanced set -o /UserVars/ESXiShellInteractiveTimeout -i 600
esxcli system settings advanced set -o /UserVars/DCUIIdleTimeout -i 900
```

### **2.5. Verify Authorized_keys is Empty**
```bash
cat /etc/ssh/keys-root/authorized_keys
```

---

## **3. Network & Security Hardening**

### **3.1. Disable Unused Protocols (TLS 1.0, 1.1)**
```bash
echo "tls.disable=1" >> /etc/vmware/config
```

### **3.2. Disable Unused Services**
```bash
chkconfig snmpd off
chkconfig sfcbd-watchdog off
chkconfig slpd off
```

### **3.3. Enable Audit Logging & Remote Log Collector**
```bash
esxcli system syslog config set --loghost=udp://192.168.1.100:514
esxcli system syslog reload
```

### **3.4. Configure vCenter Firewall**
- Configure firewall rules to restrict access to vCenter services.

---

## **4. Virtual Machine Hardening**

### **4.1. Disable Unused Hardware**
```bash
vmware-cmd /vmfs/volumes/datastore1/VM/VM.vmx setconfig bios.bootOrder "disk"
```

### **4.2. Secure VM Console Access**
```bash
vim-cmd vmsvc/getallvms
vim-cmd vmsvc/restrict-console-access VMID true
```

### **4.3. Secure Network Settings**
- Ensure **Forged Transmits, MAC Address Changes, and Promiscuous Mode** are set to **Reject** in vSwitch settings.

---

## **5. Cluster & Storage Security**

### **5.1. Enable High Availability (HA) at Cluster Level**
- Configure HA to protect workloads from host failures.

### **5.2. Configure Secure Boot & TPM for Hosts**
- Enable **UEFI Secure Boot** in BIOS.
- Ensure TPM 2.0 is enabled for integrity verification.

### **5.3. Configure Secure Storage Access**
- Enable **Bidirectional CHAP authentication** for iSCSI storage.

---

## **6. Advanced Security & Compliance**

### **6.1. Restrict Remote Console Connections**
```bash
esxcli system settings advanced set -o /UserVars/RemoteVMConsoleConnections -i 1
```

### **6.2. Secure Logging & Monitoring**
- Ensure logs are stored on a persistent datastore.
```bash
esxcli system syslog config set --logdir=/vmfs/volumes/datastore1/logs
esxcli system syslog reload
```

### **6.3. Disable Unnecessary Virtual Machine Features**
```bash
echo 'isolation.tools.copy.disable = "TRUE"' >> /etc/vmware/config
echo 'isolation.tools.paste.disable = "TRUE"' >> /etc/vmware/config
echo 'isolation.tools.dragAndDrop.disable = "TRUE"' >> /etc/vmware/config
```

### **6.4. Configure ESXi Firewall for Restricted Service Access**
- Restrict unnecessary services using `esxcli network firewall ruleset` commands.

### **6.5. Enable Lockdown Mode**
```bash
esxcli system settings advanced set -o /UserVars/SuppressShellWarning -i 1
```

### **6.6. Secure vCenter Backup & Disaster Recovery**
- Ensure vCenter backups are configured via VAMI.

### **6.7. Harden vSwitch Security Policies**
```bash
esxcli network vswitch standard policy security set -v vSwitch0 -f false -m false -p false
```

---

## **Summary**
✅ Apply all firmware, ESXi, vCenter, and VMware Tools updates.
✅ Configure strong authentication and password policies.
✅ Restrict unnecessary services and network access.
✅ Enable secure logging and compliance monitoring.
✅ Harden virtual machine security configurations.
✅ Implement cluster security measures like HA and secure boot.

---

This comprehensive guide ensures a hardened and secure VMware infrastructure. 🚀

