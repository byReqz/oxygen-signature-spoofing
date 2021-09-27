# oxygen-signature-spoofing
this repo is meant as a simple starting point for patching signature spoofing on oxygen os

# structure
## README.md
includes this structure, basic information and the how-to's

## /bin
includes tools used to patch the jar

## /build
default folder for builds to end up in, also includes prebuilt zips

# how-to
## prerequisites

- TWRP and Magisk already being present on the phone
- access to a Linux shell (WSL also works)
- ADB + drivers if required
- Java

## Android 5-9
Your system is most likely supported by NanoDroid Patcher and can be patched by flashing the magisk module: https://gitlab.com/Nanolx/NanoDroid

If manual deodexing is still needed, as for example AOSP 9, you can follow the tutorial down below:

1. Boot your phone into TWRP and mount the system
2. Pull the services.jar from your system
	- `adb pull /system/framework framework`
	- on OxygenOS you might need to pull from `/system/system/framework/` or `/system_ext/system/framework/services.jar`
3. Back up the file
	- `cp framework/services.jar services.jar.bak`
4. Determine your architecture
	- this will most likely be `arm64` but could be `arm` on older phones
5. Extract the required files with baksmali
	- `java -jar baksmali.jar x framework/oat/[arch]/services.odex -d framework/[arch] -d framework/ -o services-new`
6. Patch the extracted files with smali
	- `java -jar smali.jar a services-new -o classes.dex`
7. If it was successfully patched, add the new dex to the jar
	- `zip -j framework/services.jar classes*.dex`
8. Push the new jar to the device
	- `adb push framework/services.jar /system/framework`
	- `adb shell chmod 0644 /system/framework/services.jar`
	- `adb shell chown root:root /system/framework/services.jar`
9. Now flash NanoDroid Patcher

## Android 10
As far as I know, its not possible to patch the vdex files in OxygenOS 10.
(for generic Android 10 systems, check here: https://gitlab.com/Nanolx/NanoDroid/-/blob/master/doc/DeodexServices.md#vdex)

## Android 11
1. Boot your phone into TWRP and mount the system
2. Pull the services.jar from your system
        - `adb pull /system/framework/services.jar`
        - on OxygenOS you might need to pull from `/system/system/framework/services.jar` or `/system_ext/system/framework/services.jar`
3. Back up the file
        - `cp services.jar services.jar.bak`
4. Patch the jar using oF2pks' haystack and his custom OxygenOS hook
	- `java -jar dexpatcher-1.8.0-beta1.jar -a 30 -M -v -d -o ./ services.jar 11-hook-services.jar.dex 11core-services.jar.dex`
5. add the resulting dex files back into a new jar
	- `mkdir -p build/system/framework && cd build`
	- `zip -j system/framekwork/services.jar ../classes*.dex`
6. replace the services.jar in spoof_AVDapi30.zip
	- `cp bin/spoof_AVDapi30.zip .`
	- `zip -u spoof_AVDapi30.zip /system/framework/services.jar`
7. flash the module (build/spoof_AVDapi30.zip) through Magisk manager while booted into the system

# credits
this project was mostly inspired by https://gitlab.com/Nanolx/NanoDroid/-/issues/169 and most things here have been derived from that thread. <br>
base for the tutorial can be found here: https://forum.xda-developers.com/t/signature-spoofing-on-unsuported-android-11-r-roms.4214143/ <br>
thanks to oF2pks for the oxygen-os-hook and the tool that makes it all possible https://gitlab.com/oF2pks/haystack/-/tree/11-attempt <br>
