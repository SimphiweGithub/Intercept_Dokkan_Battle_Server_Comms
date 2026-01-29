# Dokkan Battle API Inspector: Fiddler + Nox (Android 7) + Custom Xposed + SuperSU

**Disclaimer:** The following documentation is for educational purposes only.

## Objective
To establish a stable Man-in-the-Middle (MitM) environment using **Fiddler Classic** and **Nox Emulator** that allows for the decryption and analysis of game API traffic.

## Tech Stack
* **Emulator:** Nox Player (Android 7 | 32-bit instance)
* **Proxy/Inspection:** Fiddler Classic
* **Root Framework:** Xposed Framework (SDK 24 for Android 7.0/7.1)
* **SSL Bypass:** JustTrustMe module
* **Root Management:** SuperSU
* **Utilities:** OpenSSL (Certificate Hashing)

---

## Step 1: Fiddler Configuration
**Goal:** Configure Fiddler to accept remote traffic and decrypt HTTPS.

1.  Open Fiddler Classic.
2.  Navigate to **Tools > Options > Connections**.
3.  Ensure your settings match the configuration below (Note the listening port, usually 8888):
    ![Connections Settings](images/required_connections_settings.png)
4.  Navigate to the **HTTPS** tab.
5.  Ensure "Decrypt HTTPS traffic" is checked.
6.  Click **Actions** (top right of the tab) and select **Export Root Certificate to Desktop**.
    ![HTTPS Settings](images/https_tab_settings.png)
7.  Restart Fiddler to apply changes.

## Step 2: Certificate Hashing
**Goal:** Convert Fiddler's SSL certificate into the specific hash format required by the Android System.

1.  Open the Win64 OpenSSL Command Prompt.
2.  Navigate to the directory containing your exported `FiddlerRoot.cer`.
3.  Run the following command to convert the certificate to PEM format:
    ```bash
    openssl x509 -inform DER -in FiddlerRoot.cer -out FiddlerRoot.pem
    ```
4.  Generate the subject hash required by Android:
    ```bash
    openssl x509 -inform PEM -subject_hash_old -in FiddlerRoot.pem
    ```
    * *Expected Output Hash:* `269953fb`
5.  Rename the resulting file to `269953fb.0`.

## Step 3: Emulator Configuration
1.  **Instance Creation:** Open Nox Multi-Instance Manager and create a new **Android 7 (32-bit)** instance.
    * *Note:* Android 9 or 64-bit instances are not compatible with this specific Xposed configuration.
2.  **Root Access:** Go to **System Settings > General** and enable Root.
3.  **Network Settings:** Ensure **Bridge Mode** is **Disabled** (standard NAT).

## Step 4: System Certificate Injection
**Goal:** Force Android to trust the Fiddler certificate as a "System" CA, bypassing user-certificate restrictions.

Navigate to your Nox `bin` directory (containing `nox_adb.exe`) and run the following commands. Replace `62026` with your specific instance port if different.

```bash
# 1. Push the hashed certificate to the SD card
nox_adb push "269953fb.0" /sdcard/

# 2. Enter the Android Shell
nox_adb shell

# 3. Elevate to Superuser
su

# 4. Remount the system partition as writable
mount -o rw,remount /system

# 5. Move the cert to the System Trusted Store
mv /sdcard/269953fb.0 /system/etc/security/cacerts/

# 6. Set critical permissions (Owner: Read/Write, Group/Others: Read)
chmod 644 /system/etc/security/cacerts/269953fb.0

# 7. Reboot to apply
reboot
```

## Step 5: Framework Installation (SuperSU & Xposed)
**Goal:** Install the hooking framework required to run the bypass module.

1.  **Install APKs:** Drag and drop the **SuperSU** and **Xposed Installer** APKs into the Nox window to install them.
2.  **Update Binaries:** Open the **SuperSU** app, select "New User," and update the SU binaries (select "Normal" mode).
3.  **Reboot:** Restart the instance to finalize the SU update.
4.  **Flash Xposed:**
    * Download the `Xposed.zip` included in this directory.
    * Push the **extracted** Xposed folder to the emulator using the following command:
        ```bash
        nox_adb push "C:\Path\To\xposed" /sdcard/Download/
        ```
    * Run the installation scrip(THATS IN THIS REPO) on your PC:
        

## Step 6: Security Bypass
**Goal:** Disable SSL Pinning for the target application.

1.  **Install Module:** Drag and drop `JustTrustMe.apk` into Nox to install it.
    > **Note:** If the emulator freezes for too long during installation, restart the instance. The icon may be gone from your home screen, but `JustTrustMe` will still be installed as a module.
2.  **Enable Module:** Open the **Xposed Installer** app and navigate to **Modules**.
3.  **Activate:** Check the box next to **JustTrustMe**.
4.  **Reboot:** Restart the instance to activate the module.

---
## Final results


## Operational Strategy
**Challenge:** You cannot leave the proxy and bypass active continuously, as it will break standard browser functionality and Google Play services. You must switch between two modes.

### Mode A: Capture Mode (Analysis)
**Use Case:** Inspecting game traffic.
1.  **Xposed:** Enable `JustTrustMe` -> **Reboot**.
2.  **Fiddler:** Open Fiddler on PC.
3.  **Android Wi-Fi:** Set Proxy to **Manual**.
    * **Host:** `192.168.10.18` (Your PC's Local IP).
    * **Port:** `8888`.
4.  **Status:** Game traffic is visible. Browser will fail (`net::ERR_FAILED`).

### Mode B: Normal Mode (Maintenance)
**Use Case:** Downloading game data, updates, or using the Play Store.
1.  **Xposed:** Disable `JustTrustMe` -> **Reboot**.
2.  **Android Wi-Fi:** Set Proxy to **None**.
3.  **Status:** Browser and Play Store function normally. Game traffic is encrypted.

---

## Troubleshooting

### Issue 1: The "Orphan Proxy" (net::ERR_FAILED in browser)
**Symptom:** Fiddler is closed, but the Android browser still has no connectivity.
**Cause:** Android retains the proxy setting in the global table or the specific Wi-Fi profile is stuck.

**Solution 1 (UI Fix):**
Go to **Settings > Wi-Fi > WiredSSID (Long Press) > Modify Network**. Manually switch Proxy from **Manual** to **None**.

**Solution 2 (ADB Force Clear):**
```bash
nox_adb shell settings put global http_proxy :0
nox_adb shell settings delete global http_proxy
nox_adb shell settings delete global global_http_proxy_host
nox_adb shell settings delete global global_http_proxy_port
nox_adb shell am broadcast -a android.intent.action.PROXY_CHANGE
```
### Credits & Licenses

SSL Bypass: JustTrustMe by Fuzion24 (Apache 2.0).<br/>
Xposed Script: Modified from Install-Xposed-on-Memu by SoheilShakib.<br/>
Framework: Xposed by rovo89.<br/>
