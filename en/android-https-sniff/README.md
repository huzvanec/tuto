# How to sniff full HTTPS traffic on Android 7.0+
# The main problem
Starting with Android 7.0 (Nougat), [apps by default no longer trust user-installed CA certificates](https://android-developers.googleblog.com/2016/07/changes-to-trusted-certificate.html).
This makes sniffing HTTPS traffic much harder.

# The solutions
There are basically two workarounds, and neither is particularly elegant:
### 1. [Installing the CA certificate as system (the bad approach)](system-cert.md)
Create an emulator, root it, install the certificate and convince the emulator that it's a system certificate.
### 2. [Modify the APKs (the even worse approach)](modify-apks.md)
Decompile the APKs, modify them to accept user certificates, compile them again and sign them using a custom keystore.  
I spent hours trying this method first-and it ultimately didn’t end up working.