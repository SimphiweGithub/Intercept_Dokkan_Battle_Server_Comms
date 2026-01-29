# Dokkan Battle API Inspector: Fiddler + Nox (Android 7) + Custom Exposed .apk + SuperSu Setup

The following is for educational purposes.

# What this is?
Its a  "Master Log" documenting the procedure to bypass SSL Pinning and inspect HTTPS traffic in **Dragon Ball Z: Dokkan Battle (Global)**.

# Objective
To establish a stable Man-in-the-Middle (MitM) environment using **Fiddler Classic** and **Nox Emulator** that allows for the decryption and analysis of game API traffic .

# Tech Stack
* **Emulator:** Nox (Android 7 | 32-bit)
* **Proxy/Inspection:** Fiddler Classic
* **Root Framework:** Xposed (SDK 24)
* **SSL Bypass:** JustTrustMe module
* **SuperSU:** Help root management framework
* **OpenSSL:** Hash certificate into acceptable format


# Step 1
Goal: Setup Fiddler for this operation
-Open Fiddler Classic<br/>
-Then go to tools then Connections and make sure you select the same options I have<br/>.
-Remember that port number<br/>
![Alt text for the image](images/required_connections_settings.png)

<br/>-Then go to the HTTPS tab and make sure it looks like the following screenshot and then go to Actions in the top right of this tab and click the export root certificater button <br/>
![Alt text for the image](images/https_tab_settings.png)
<br/>-Restart Fiddler

# Step 2 
Goal: Convert Fiddler’s SSL certificate into a hash format Android System accepts.<br/>
<br/>-Now open Win64 OpenSSL Command Prompt
<br/>-cd over to to your desktop directory or where ever you stored the certificate
<br/>-Then run the following commands

1.  Convert the certificate to PEM format using OpenSSL:
    ```bash
    openssl x509 -inform DER -in FiddlerRoot.cer -out FiddlerRoot.pem
    ```
2.  Generate the specific hash required by Android:
    ```bash
    openssl x509 -inform PEM -subject_hash_old -in FiddlerRoot.pem
    ```
    * *Resulting Hash:* `269953fb`
3.  Rename the file to `269953fb.0`.

# Step 3: Emulator Configuration
1.  **Instance Creation:** Create a new **Android 7 (32-bit)** instance in Nox Multi-Instance Manager.
    * *Critical Constraint:* Do NOT use Android 9 or 64-bit instances; the Xposed framework will fail.
2.  **Root Access:** Enable Root in **System Settings > General**.
3.  **Network Settings:** Ensure **Bridge Mode** is **Disabled** (use standard NAT).


### Phase 3: System Certificate Injection
*Goal: Force Android to trust Fiddler as a "System" CA, bypassing user-cert restrictions.*

Run the following commands in your ADB terminal (Example Port: `62026`):

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

