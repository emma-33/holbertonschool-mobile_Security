## Installation commands of emulator and frida
adb -s emulator-5554 install task3_d.apk
adb push frida-server-17.5.1-android-x86_64 /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server
adb shell /data/local/tmp/frida-server &

list_classes.js :
Java.perform(() => {
    Java.enumerateLoadedClasses({
        onMatch: function(className) {
            if (className.includes("holberton"))
                console.log(className);
        },
        onComplete: function() {
            console.log("Done listing classes.");
        }
    });
});

frida -U -p 21668 -l list_classes.js

...                                                         
com.holberton.task4_d.MainActivity$onCreate$1
com.holberton.task4_d.MainActivity
com.holberton.task4_d.MainActivity$onCreate$1$1
com.holberton.task4_d.ComposableSingletons$MainActivityKt
com.holberton.task4_d.MainActivityKt
com.holberton.task4_d.MainActivity$onCreate$1$1$1
com.holberton.task4_d.ui.theme.ColorKt
com.holberton.task4_d.ui.theme.TypeKt
com.holberton.task4_d.MainActivityKt$GreetingPreview$1
com.holberton.task4_d.ui.theme.ThemeKt$Task4_dTheme$1
com.holberton.task4_d.ui.theme.ThemeKt$Task4_dTheme$2
com.holberton.task4_d.ui.theme.ThemeKt
com.holberton.task4_d.MainActivity$retrieveEncryptedData$1
com.holberton.task4_d.MainActivityKt$Greeting$2
Done listing classes.
  


dump_methods.js :
Java.perform(() => {
    const target = "com.holberton.task4_d.MainActivity$retrieveEncryptedData$1";
    const clazz = Java.use(target);

    console.log("Methods in " + target + ":");
    clazz.class.getDeclaredMethods().forEach(method => {
        console.log("  -> " + method.toString());
    });
});
frida -U -p 27890 -l dump_methods.js 
      
Methods in com.holberton.task4_d.MainActivity$retrieveEncryptedData$1:
  -> public final kotlin.coroutines.Continuation com.holberton.task4_d.MainActivity$retrieveEncryptedData$1.create(java.lang.Object,kotlin.coroutines.Continuation)
  -> public java.lang.Object com.holberton.task4_d.MainActivity$retrieveEncryptedData$1.invoke(java.lang.Object,java.lang.Object)
  -> public final java.lang.Object com.holberton.task4_d.MainActivity$retrieveEncryptedData$1.invoke(kotlinx.coroutines.CoroutineScope,kotlin.coroutines.Continuation)
  -> public final java.lang.Object com.holberton.task4_d.MainActivity$retrieveEncryptedData$1.invokeSuspend(java.lang.Object)

                                                 
In MainActivityKt, found with jadx a suspicious function aBcDeFgHiJkLmNoPqRsTuVwXyZ123456. It's not called by retrieveEncryptedData so we have to call it manually.

flag.js : 
Java.perform(() => {
    console.log("[*] Waiting for MainActivityKt to load...");

    Java.enumerateLoadedClasses({
        onMatch: function (c) {
            if (c.includes("MainActivityKt")) {
                console.log("[*] Found class:", c);
            }
        },
        onComplete: function () {
            console.log("[*] Done scanning.");
        }
    });

    const Class = Java.use("java.lang.Class");
    const MainActivityKt = Java.use("com.holberton.task4_d.MainActivityKt");

    // Get java.lang.Class object
    const clazz = MainActivityKt.class;

    // Get the private static method using reflection
    const methods = clazz.getDeclaredMethods();
    let target = null;

    for (let i = 0; i < methods.length; i++) {
        let name = methods[i].getName();
        if (name.startsWith("aBcDeFgHiJkLmNoPqRsTuVwXyZ123456")) {
            target = methods[i];
            break;
        }
    }

    if (!target) {
        console.log("[-] Could not find decoding function!");
        return;
    }

    console.log("[*] Found method:", target);

    // Allow access
    target.setAccessible(true);

    // Dummy callback = Kotlin Function1<String, Unit>
    const Callback = Java.registerClass({
        name: 'com.example.DummyCallback2',
        implements: [Java.use('kotlin.jvm.functions.Function1')],
        methods: {
            invoke: function (flag) {
                console.log("\n===== FLAG DECODED =====");
                console.log(flag);
                console.log("========================\n");
                return null;
            }
        }
    });

    // Invoke static method (null because it is static)
    console.log("[*] Invoking decoder…");
    target.invoke(null, [Callback.$new()]);
});

frida -U -p 27890 -l flag.js
...
[*] Waiting for MainActivityKt to load...
[*] Found class: com.holberton.task4_d.ComposableSingletons$MainActivityKt
[*] Found class: com.holberton.task4_d.MainActivityKt
[*] Found class: com.holberton.task4_d.MainActivityKt$GreetingPreview$1
[*] Found class: com.holberton.task4_d.MainActivityKt$Greeting$2
[*] Done scanning.
[*] Found method: private static final void com.holberton.task4_d.MainActivityKt.aBcDeFgHiJkLmNoPqRsTuVwXyZ123456(kotlin.jvm.functions.Function1)
[*] Invoking decoder…

===== FLAG DECODED =====
Holberton{calling_uncalled_functions_is_now_known!}
========================

[Android Emulator 5554::PID::27890 ]->

