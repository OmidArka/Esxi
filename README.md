OMID ARKA
```bash

# **ESXi & vCenter Hardening Guide**

## **1. Hardening ESXi**

### **1.1. Update & Patch Management**
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

---

### **1.2. BIOS/UEFI & Hardware Security**
- Enable **Secure Boot**, **VT-d**, and **Intel TXT** in BIOS.

---

### **1.3. Restrict Access**
#### **Disable SSH & ESXi Shell:**
```bash
esxcli network firewall ruleset set -r sshServer -e false
esxcli system settings advanced set -o /UserVars/ESXiShellTimeOut -i 0
```
#### **Enable Lockdown Mode:**
```bash
esxcli system settings advanced set -o /UserVars/SuppressShellWarning -i 1
```
#### **Check Lockdown Mode Status:**
```bash
esxcli system settings advanced list | grep -i lockdown
```

---

### **1.4. Configure ESXi Firewall**
#### **Check Firewall Status:**
```bash
esxcli network firewall get
```
#### **List Firewall Rules:**
```bash
esxcli network firewall ruleset list
```
#### **Disable Unnecessary Services (e.g., SNMP):**
```bash
esxcli network firewall ruleset set -r snmpServer -e false
```

---

### **1.5. User Management**
#### **Change Root Password:**
```bash
passwd root
```
#### **Create a New Admin User & Disable Root SSH:**
```bash
esxcli system account add --id=adminuser --password=StrongPass! --description="Admin User"
```

---

### **1.6. Logging & Monitoring**
#### **Check Syslog Configuration:**
```bash
esxcli system syslog config get
```
#### **Set Remote Syslog Server:**
```bash
esxcli system syslog config set --loghost=udp://192.168.1.100:514
```
#### **Restart Syslog Service:**
```bash
esxcli system syslog reload
```

---

## **2. Hardening vCenter**

### **2.1. Update & Patch Management**
#### **Update via VAMI:**
Access:
```plaintext
https://vcenter-ip:5480
```
Check & install updates under **Update** section.

---

### **2.2. User Management & Access Control**
#### **Check Existing Users & Roles:**
```bash
Get-VIPermission | Select Principal, Role, Entity
```
#### **Remove Unnecessary Users:**
```bash
Remove-VIPermission -Principal "olduser"
```
#### **Use SSO Authentication** instead of local accounts.

---

### **2.3. Enable Multi-Factor Authentication (MFA)**
Enable MFA under **Access** in VAMI.
Use **DUO, RSA, or VMware Verify**.

---

### **2.4. Firewall & Network Security**
#### **Check Firewall Status:**
```bash
esxcli network firewall get
```
#### **Restrict Unnecessary Ports** in vCenter settings.

---

### **2.5. Encrypt Virtual Machines**
1. Open vSphere.
2. Navigate to VM settings.
3. Enable **VM Encryption** under **Security Settings**.

---

## **3. Security Auditing & Compliance**
1. Run **VMware Security Advisor** for vulnerability assessment.
2. Use **CIS Benchmark for VMware** to verify security configurations.
3. Monitor logs and security events continuously.

---

### **Summary**
✅ Disable unnecessary services.
✅ Enable MFA authentication.
✅ Apply regular updates.
✅ Send logs to a remote server.
✅ Encrypt virtual machines.

---

This guide ensures a hardened and secure VMware infrastructure. 🚀

