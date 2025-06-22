# 🔐 Resetting the Administrator Password in vCenter

If you've forgotten the password for the **Administrator@vsphere.local** account in vCenter, don't worry. As long as you have console access to the vCenter Server and know the `root` password, you can easily reset the administrator password and regain access.

Follow these steps:

1. Download and install [PuTTY](https://www.putty.org).  
   Launch PuTTY and enter your vCenter Server's IP address or hostname in the **Host Name (or IP address)** field.

2. Log in using the `root` user credentials.  
   If SSH is disabled, enable it via the direct console by running the following commands:

   ```bash
   Shell.set --enabled true
   shell
   ```

3. Once in the shell, run the following tool:

   ```bash
   /usr/lib/vmware-vmdir/bin/vdcadmintool
   ```

4. In the menu, type `3` to select the **Reset account password** option.

5. Enter the full username for the account you want to reset. Typically, this will be:

   ```
   Administrator@vsphere.local
   ```

6. Press Enter. A new randomly generated password will be displayed.

You can now log in to the vCenter Server using this newly generated password and continue managing your environment.

**Security Tip:**  
After logging in, it is highly recommended to change the password to one of your choosing and ensure the `root` account remains secure and access-controlled.

> This procedure has been tested and verified to work with vCenter versions 6.7 and above.

---

![Reset vCenter Password](http://techtik.com/wp-content/uploads/2019/05/Reset-Password-vCenter-9.png)
