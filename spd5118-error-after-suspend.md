# Blacklisting `spd5118` Module to Fix Suspend/Resume Issues

## Problem Statement

On my MSI CreatorPro 16 AI Studio laptop running Ubuntu 24.10, the system would fail to resume from suspend. Upon inspection of the kernel logs, I observed the following error:

```
spd5118 1-005c: Failed to write b: 0 = -6
PM: dpm_run_callback(): spd5118_resume [spd5118] returns -6
PM: failed to resume async: error -6
```

The issue was traced to the `spd5118` module, which appeared to cause a problem during the power state transitions.

## Solution

To resolve this issue, I blacklisted the `spd5118` module to prevent it from loading.

### Steps to Blacklist `spd5118`

1. **Create a Blacklist Configuration File:**
   Create a new configuration file for blacklisting the module in the `/etc/modprobe.d/` directory:

   ```bash
   sudo nano /etc/modprobe.d/blacklist-spd5118.conf
   ```

2. **Add the Blacklist Entry:**
   Add the following line to the file:

   ```
   blacklist spd5118
   ```

3. **Update Initramfs:**
   Regenerate the initramfs to apply the changes:

   ```bash
   sudo update-initramfs -u
   ```

4. **Reboot the System:**
   Reboot the system to ensure the `spd5118` module does not load:

   ```bash
   sudo reboot
   ```

5. **Verify the Module is Blacklisted:**
   After the reboot, check if the `spd5118` module is loaded:

   ```bash
   lsmod | grep spd5118
   ```

   If no output is returned, the module has been successfully blacklisted.

## Outcome

After blacklisting the `spd5118` module, the suspend/resume functionality of the laptop works without any issues. The error no longer appears in the logs.
