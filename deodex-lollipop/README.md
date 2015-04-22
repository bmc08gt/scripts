# Lollipop deodexing

######Deodexing apk's on lollipop is almost as simple as API < 21 and only invoves two extra steps.

The workflow for the new deodex process is as follows below. The <arch> is the architecture for the apk being deodexed.
  1. `oat2dex boot system/framework/<arch>/boot.oat`
  2. `oat2dex <path/to/app/<arch>/app.odex>` `system/framework/<arch>/odex`
  3. baksmali as normal
  4. smali as normal
  

######The actual behind-the-scenes info

ART changes things with its 'Ahead of Time Compiling'. Android creates a boot.oat on boot containing the classpath jars that apks
link against for functionality. Before these the .odex for the classpath was compatible with baksmali/smali, but due to changes with
ART vs Dalvik this is no longer the case now.  The classpath needs to extracted from the boot.oat. The resulting dex/ and odex/ are then
used to create a bootclasspath that can be used to create a baksmali/smali compatible odex, by essentially deoptimizing it.  Once 
the compatible odex is created, baksmali and smali can be used like normal - even allowing smali edits to exisiting APKs.

######A working (manual) example

To get started you will need to pull the framework directory off the device.

    mkdir framework
    cd framework
    adb pull system/framework

The next step is to pull the app's folder from system/app or system/priv-app.
For this example I will be using SecSettings2.apk from a Samsung Galaxy S6.

    mkdir SecSettings2
    cd SecSettings2
    adb pull /system/priv-app/SecSettings2

The next step is to determine the architecture of the apk to be deodexed. This is done checking whether the directory that the 
apk's folder contents were extracted to contains an arm or arm64 folder (32-bit or 64-bit, respectively).
The SecSettings2.apk is 64-bit as its folder contains an arm64 folder.

Once the architecture is determined, the next step is to extract the classpath from boot.oat. Replace arm64 with arm below
if your app is only 32-bit.

    oat2dex boot framework/arm64/boot.oat

This will extract the classpath from the boot.oat and place the dex and odex into separate directories in framework/<arch>.

Now the classpath can be used to create a compatible odex that baksmali and smali is compatible with. Note that the result odex
will have a .dex extension. This is NOT to be placed into the apk as the app will NOT work.

    oat2dex SecSettings2/arm64/SecSettings2.odex framework/arm64/odex

We now have a odex (labled SecSettings2.dex in SecSettings2/) that is compatible with baksmali and smali.

To baksmali the odex:

    baksmali -a 21 -x SecSettings2.dex -d framework -o smali/SecSettings2
    
To deodex the smali into a dex:

    smali smali/SecSettings2
    
The last command outputs a classes.dex that is going to go into the apk.

    zip -gjq SecSettings2.apk classes.dex

Now we need to remove the arm or arm64 folder from the apk's location on the device.

    adb shell su -c 'mount -o remount,rw /system'
    adb shell su -c 'rm -rf system/priv-app/SecSettings2/arm64'

Now we simply need to push the apk to the device, fix permissions, and reboot.

    adb push SecSettings2.apk /data/local/tmp/
    adb shell su -c 'dd if=/data/local/tmp/SecSettings2.apk of=/system/priv-app/SecSettings2/SecSettings2.apk'
    adb shell su -c 'chmod 644 /system/priv-app/SecSettings2/SecSettings2.apk'
    adb reboot (You may need to wipe data depending on the app being deodexed)
    
That's it. Happy modding :)

######But I have made it easy for ya.....

deodex-app makes it stupid simple to deodex an apk from the device by running one command

    ./deodex-app SecSettings2.apk
    
That's it. The script will determine the apk's location in system, check if it's already deodexed, pull the neccessary
files to perform the deodexing, and will push the apk back to the device and reboot.

/lazydev

Huge credits to @JesusFreke for baksmali and smali, and credit to @svadev for the updated oat2dex
