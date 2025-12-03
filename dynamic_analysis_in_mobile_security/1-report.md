# Frida Native Function Hooking Report

## Installation commands of emulator and frida
adb -s emulator-5554 install task1_d.apk
adb push frida-server-17.5.1-android-x86_64 /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server
adb shell /data/local/tmp/frida-server &

// Launch emulator in android studio and launch app

## Library
Found in Android Studio editor the name of the external library is "libnative-lib.so".

## Script (script2.js)
Java.perform(function () {
    console.log("[+] Hooking MainActivity.getSecretMessage()");

    var MainActivity = Java.use("com.holberton.task2_d.MainActivity");

    MainActivity.getSecretMessage.implementation = function () {
        console.log("[+] getSecretMessage() called");

 
        var result = this.getSecretMessage();

        if (result && result.indexOf("Holberton{") !== -1) {
            console.log("\n===== FLAG FOUND =====");
            console.log(result);
            console.log("======================\n");
        }

        return result;
    };
});

## Script execution with frida
frida -U -f com.holberton.task2_d -l script.js

// Click on the button in the app and get the result in frida:

Spawned `com.holberton.task2_d`. Resuming main thread!                  
[Android Emulator 5554::com.holberton.task2_d ]-> [+] Hooking MainActivity.getSecretMessage()
[+] getSecretMessage() called

===== FLAG FOUND =====
Holberton{native_hooking_is_no_different_at_all}
======================

