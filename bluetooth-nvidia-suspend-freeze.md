# Fixing Bluetooth Suspend/Resume Issue on MSI Creator 16 AI Studio

## Problem Description
When using the MSI Creator 16 AI Studio laptop on Ubuntu 24.10 with i3wm, the system would freeze or behave erratically after suspend/resume cycles when Bluetooth was enabled. The issue was especially pronounced when using a Bluetooth mouse, with symptoms including:

- Complete system freeze during suspend/resume when NVIDIA drivers were active.
- Occasional stuttering or lag with the Bluetooth mouse during GPU-intensive operations.

Disabling Bluetooth entirely resolved the problem, indicating a conflict between Bluetooth and NVIDIA GPU during the suspend/resume process.

## Root Cause
The issue appears to be a conflict between the Bluetooth stack and the NVIDIA driver during the suspend process. Disabling Bluetooth early in the suspend cycle prevents this conflict.

## Solution
To address the problem, we created two custom `systemd` services to:

1. Disable Bluetooth at the start of the suspend cycle.
2. Re-enable Bluetooth after the resume cycle, only if it was enabled before suspension.

### Steps to Implement

#### 1. Create a Service to Disable Bluetooth
Create a new `systemd` service to turn off Bluetooth before the system enters suspend.

File: `/etc/systemd/system/bluetooth-disable.service`

```ini
[Unit]
Description=Disable Bluetooth before sleep
Before=sleep.target
StopWhenUnneeded=yes

[Service]
Type=oneshot
ExecStart=/bin/bash -c "bluetoothctl power off; sleep 1"
RemainAfterExit=yes

[Install]
WantedBy=sleep.target
```

#### 2. Create a Service to Enable Bluetooth
Create a complementary service to restore Bluetooth after the system resumes.

File: `/etc/systemd/system/bluetooth-enable.service`

```ini
[Unit]
Description=Enable Bluetooth after sleep
After=suspend.target

[Service]
Type=oneshot
ExecStart=/bin/bash -c "bluetoothctl power on; sleep 1"

[Install]
WantedBy=suspend.target
```

#### 3. Reload and Enable the Services
After creating these files, reload `systemd` and enable the services:

```bash
sudo systemctl daemon-reload
sudo systemctl enable bluetooth-disable.service
sudo systemctl enable bluetooth-enable.service
```

#### 4. Testing
1. Manually test the services to ensure they work as expected:
   ```bash
   sudo systemctl start bluetooth-disable.service
   sudo systemctl start bluetooth-enable.service
   ```

2. Suspend and resume the system:
   ```bash
   sudo systemctl suspend
   ```

3. Verify that Bluetooth is disabled before suspend and re-enabled after resume.

### Debugging and Logs
For troubleshooting, you can log actions to a file:

- Add to `bluetooth-disable.service`:
  ```ini
  ExecStart=/bin/bash -c "bluetoothctl power off; echo 'Bluetooth OFF at $(date)' >> /tmp/bluetooth.log; sleep 1"
  ```

- Add to `bluetooth-enable.service`:
  ```ini
  ExecStart=/bin/bash -c "bluetoothctl power on; echo 'Bluetooth ON at $(date)' >> /tmp/bluetooth.log; sleep 1"
  ```

Logs can be reviewed using:
```bash
cat /tmp/bluetooth.log
```

## Results
After implementing these changes:

- The system no longer freezes during suspend/resume cycles.
- The Bluetooth mouse operates normally after resuming.
- No conflicts occur with the NVIDIA GPU drivers.

This fix ensures stable operation of the MSI Creator 16 AI Studio laptop with Bluetooth and NVIDIA drivers enabled.
