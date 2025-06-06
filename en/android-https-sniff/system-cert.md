# Installing the CA certificate as system
Create an emulator, root it, install the certificate and convince the emulator that it's a system certificate.
# 0 Requirements
- [Android Studio](https://developer.android.com/studio) (or at least the emulator)
- [Git](https://git-scm.com/downloads)
- [Burp](https://portswigger.net/burp/releases) (or some other debugging proxy)
# 1 Rooting the emulated device
The device needs to be rooted to modify the system partition (and installing the certificate). This approach **does** work with Play Store images. Use the `API-31 "S" (Android 12.0)` system image for the best results.
### 1.1.Install [rootAVD](https://gitlab.com/newbit/rootAVD)
A utility that helps with rooting the emulated device using [Magisk](https://github.com/topjohnwu/Magisk).
```bash
git clone https://gitlab.com/newbit/rootAVD.git && cd rootAVD
```
### 1.2 Add android platform tools to path
**⚠️ From now on, you must run all of the commands in the same terminal! ⚠️**
```bash
export PATH=$PATH:$HOME/Android/Sdk/platform-tools
```
### 1.3 Start the AVD
### 1.4 List all AVDs
```bash
./rootAVD ListAllAVDs
```
### 1.5 Root the AVD
The previous command listed some suggestions, copy the one looking like this:
```
./rootAVD.sh system-images/android-<your_avd_api_version>/google_apis_playstore/x86_64/ramdisk.img
```
and run it. When asked about the Magisk version, choose 1 (local stable). If everything works correctly, the emulator should shut down.
### 1.6 Initialize Magisk
Start your AVD again and open the newly installed Magisk app. The app will prompt you with a window titled `Requires Additional Setup`. Choose `OK` and wait for your device to reboot.

Then grant root access to the console:
```bash
adb shell "su"
```
A popup will appear in the emulator requesting root access. Grant it.
# 2 Installing the CA certificate
### 2.1 Configure Burp
Start Burp, go to `Burp | Settings | Tools | Proxy` and edit the default proxy:
`Bind to address`: `All interfaces`
### 2.2 Export the certificate
Export the CA certificate from your debugging proxy app as `PEM` or `DER` and save it on disk.
(In Burp it's `Burp | Settings | Tools | Proxy | Import / Export CA certificate | Export | Certificate in DER format`)
### 2.3 Install the certificate (the normal way)
First push the certificate file to the emulator.
```bash
adb push /path/to/your/certificate.der /sdcard/Download
```
And then install the certificate (`Settings | Search | "CA certificate"`...)
### 2.4 Make a system override module
Since Android 10.0, `/system` is mounted as read-only, that mean's that the certificate can't be just easily copied there. An adb module needs to be created and its contents will be automatically copied to `/system` during boot.
```bash
adb shell
su
mkdir -p /data/adb/modules/certmodule/system/etc/security/cacerts
mv /data/misc/user/0/cacerts-added/* /data/adb/modules/certmodule/system/etc/security/cacerts/
reboot
```
### 2.5 Enable the proxy
Go to `Emulator menu | ... | Settings | Proxy`, enable `Manual proxy configuration` and configure your proxy.

# Done!
Now all the traffic of the AVD should be going trough your proxy.