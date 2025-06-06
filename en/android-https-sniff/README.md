# How to sniff full HTTPS traffic on Android 7.0+
# The main problem
The main problem with sniffing HTTPS traffic is that [in modern versions of Android (7.0+) all applications by default don't trust user-installed CA certificates](https://android-developers.googleblog.com/2016/07/changes-to-trusted-certificate.html).

# The solution
There are basically only two ways to solve the problem
### 1. [Installing the CA certificate as system (the bad approach)](https://github.com/huzvanec/tutorials/blob/main/android-https-sniff-en/system-cert.md)
Create an emulator, root it, install the certificate and convince the emulator that it's a system certificate.
### 2. [Modify the APKs (the even worse approach)](https://github.com/huzvanec/tutorials/blob/main/android-https-sniff-en/modify-apks.md)
Decompile the APKs, modify them to accept user certificates, compile them again and sign them using a custom keystore.<br>
This is a bad approach. Been there, tried it, not fun, did not work well.