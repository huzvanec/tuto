# Modifying the APKs
Decompile the APKs, modify them to accept user certificates, compile them again and sign them using a custom keystore.
<br><br>
This is a bad approach. Been there, tried it, not fun, did not work well. But here is some useful stuff to get you started:
- If it's a `xapk`, rename it to zip and extract the apks from the inside.
- Decompile using [apktool](https://apktool.org/) (`apktool d myapp.apk`)
- If you need to read the code of the app, run [dex2jar](https://github.com/pxb1988/dex2jar) on the apk and then throw the output jar to some [java decompiler](http://www.javadecompilers.com/).
- `AndroidManifest.xml`:
```
...
<application android:networkSecurityConfig="@xml/network_security_config"
             ... >
...
```
- `res/xml/network_security_config.xml`:
```
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config>
        <trust-anchors>
            <certificates src="user"/>
            <certificates src="system"/>
        </trust-anchors>
    </base-config>
</network-security-config>
```
- Compile using apktool (`apktool b myappfolder`)
- Generate a keystore (only once, used for singing processes)
```bash
keytool -genkey -v -keystore my.keystore -keyalg RSA -keysize 2048 -validity 10000 -alias my_alias
```
- Sign the apk
```bash
apksigner sign --ks-key-alias my_alias --ks my.keystore myapp.apk
```
