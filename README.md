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
Goal: Convert Fiddler’s SSL certificate into a hash format Android System accepts.<br/>
-Install Fiddler Classic<br/>
-Then go to tools then Connections and make sure you select the same options I have.
![Alt text for the image](images/required_connections_settings.png)

<br/>-Then go to the HTTPS tab and make sure it looks like the following screenshot and then go to Actions in the top right of this tab and click the export root certificater button <br/>
![Alt text for the image](images/https_tab_settings.png)


