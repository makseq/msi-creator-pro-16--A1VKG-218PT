### **1. Configure Fingerprints**

#### **Install the Required Software**
1. Install the `fprintd` and `libpam-fprintd` packages:
   ```bash
   sudo apt update
   sudo apt install fprintd libpam-fprintd
   ```

2. Verify if your fingerprint reader is detected:
   ```bash
   lsusb
   ```
   Look for entries related to "Goodix" or other fingerprint devices.

#### **Enroll a Fingerprint**
1. Use the following command to enroll a fingerprint:
   ```bash
   fprintd-enroll
   ```
   Follow the on-screen instructions to scan your finger.

---

### **2. Add/Remove Fingerprints Including Names**
#### **Add Another Finger**
1. To specify a username or device explicitly:
   ```bash
   fprintd-enroll -f <finger>
   ```
   Replace `<finger>` with names like `right-index-finger`, `left-thumb`, etc.

#### **List Enrolled Fingerprints**
1. Check enrolled fingerprints:
   ```bash
   fprintd-list <username>
   ```

#### **Remove All Fingerprints**
1. To delete all fingerprints for a user:
   ```bash
   fprintd-delete <username>
   ```

---

### **3. Add Fingerprint to `sudo` Password**
1. Open the PAM configuration for `sudo`:
   ```bash
   sudo nano /etc/pam.d/sudo
   ```

2. Add the following line at the top:
   ```plaintext
   auth sufficient pam_fprintd.so
   ```

3. Save and exit the file. This allows fingerprint authentication for `sudo` operations.

---

### **4. Add Fingerprint to Login Screen**
#### **Enable Fingerprint for Login**
1. Open the PAM configuration for login:
   ```bash
   sudo nano /etc/pam.d/common-auth
   ```

2. Add or modify the following lines:
   ```plaintext
   auth sufficient pam_fprintd.so
   auth [success=1 default=ignore] pam_unix.so nullok try_first_pass
   auth requisite pam_deny.so
   auth required pam_permit.so
   ```

3. Save and exit.

4. Enable the fingerprint feature using Gnome or LightDM configuration if required:
   ```bash
   sudo pam-auth-update
   ```

5. Choose fingerprint authentication from the menu.

---

### **5. Test and Debug**
1. **Test sudo**:
   ```bash
   sudo ls
   ```
   Use your fingerprint to authenticate.

2. **Test Login**:
   Lock the screen or restart the system and try logging in using the fingerprint scanner.
