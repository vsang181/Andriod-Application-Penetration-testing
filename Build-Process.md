# Build Process

Understanding how Android Studio compiles code and resources into a functional Android application helps in grasping how all components fit together. This knowledge also sheds light on the protection mechanisms employed to ensure the authenticity of applications and the circumstances that can compromise their integrity.

## Compiling Apps

The process of building Android apps involves three major steps: `compilation`, `packaging`, and finally `signing`.

To begin with, you can build Android apps either for testing/debugging purposes or for release on Google Play (or other marketplaces). The primary difference between these two options is in the signing of the applications.

The build process starts with the Android Asset Packaging Tool (aapt), which is used to compile the application resources.

![yCaIB](https://github.com/vsang181/Andriod-Application-Penetration-testing/assets/28651683/0e147fc6-8b2f-480f-9bc1-272b5998a282)

This step produces the `R.java` file, which your code uses to reference the resources. The `R.java` file is essentially a bridge between your resources (like images, layouts, and strings) and the Java code, allowing you to access them programmatically.

Next, the process compiles `.java` and `.aidl` (Android Interface Definition Language) files into `.class` files. The `.java` files contain your application’s logic, while `.aidl` files are used to define interfaces for inter-process communication.

These `.class` files are then compiled, along with any third-party libraries your app might be using, into a `classes.dex` file. The `.dex` file stands for Dalvik Executable, which is the format used by the Dalvik and ART (Android Runtime) virtual machines to run your code on an Android device.

Following this, your `classes.dex` file, compiled resources, and other assets are run through the `apkbuilder` to produce an Android Package (`.apk` file). The `.apk` file is the final packaged application file that can be installed on Android devices.

The next step involves signing the `.apk` file, but for now, let’s focus on understanding the `.apk` file itself. The `.apk` file contains all the compiled code, resources, assets, and manifest file, and it is what gets distributed and installed on user devices.

By understanding this build process, you gain insights into the steps involved in transforming your source code and resources into a distributable and executable Android application, and how each step plays a role in ensuring the app's integrity and security.

## APK Structure

When Android applications are compiled, the resulting output is an Android Package (APK) file. It is a compressed archive containing all the resources necessary to run the application, including both the code and resources such as images.

To inspect the contents of an APK, you first need to change its extension to .zip and then decompress it. This can be done with any tool capable of opening ordinary zip archives.

Once decompressed and examined, the APK contents will include the following files and directories:

- AndroidManifest.xml
- classes.dex
- resources.arsc
- /assets
- /lib
- /META-INF
- /res
- Third-party libraries

### AndroidManifest.xml

This file is crucial as it dictates many security-related features, such as which version of Android the application will run on, which apps can interact with various components of the app, and what permissions they require. It also determines whether data can be accessed via USB, among other things.

The `AndroidManifest.xml` file is a binary file in the APK, which is not human-readable. To view it properly, it needs to be converted to a human-readable XML format.

Below is an example of an `AndroidManifest.xml` file from a test app:

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.appbrain.example"
    android:versionCode="30"
    android:versionName="5.10" >

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

    <application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:allowBackup="true">
        <activity
            android:label="@string/app_name"
            android:name=".ExampleActivity"
            android:exported="true">
            <intent-filter >
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity android:name=".BannerActivity" />
        <activity android:name=".ListAdsActivity" />
	<!-- Necessary AppBrain SDK entries are imported through the aar from gradle -->
    </application>
</manifest>
```

In this example, the `<package>` element shows that the app is named `com.appbrain.example`. The `<uses-sdk>` element contains the `minSdkVersion` setting, which tells the oldest version of the Android operating system the app will run on, and the `targetSdkVersion`, which determines whether to enable compatibility features when running on newer versions.

Supporting older OS versions can make the app vulnerable to known exploits in those versions. The Android ecosystem includes numerous devices from various manufacturers, many of which run customized software. This fragmentation means that some users might never receive updates to more secure versions of the OS, increasing the risk of exploitation.

Other elements in the manifest, such as `<uses-permission>`, `<activity>`, `<action>`, `<category>`, and `<intent-filter`>, define the app's permissions, components, and their interactions.

### Classes.dex

The Java code written in Android Studio is compiled into a `.dex` file (Dalvik Executable), which is used by both the older Dalvik VM and the newer Android Runtime (ART). A single `.dex` file has a limit of 65,536 methods. Most programs have one `.dex` file, but there are ways to configure support for multiple `.dex` files depending on the Android version supported.

Although looking at the `classes.dex` file in binary format is not useful as-is, it is possible to edit this file and repackage it to run modified code. This file contains the majority of the compiled application bytecode, meaning an attacker can get a copy of the code, even though it might be hard to read.

Key principles of application security:

- Client application code is not secret.
- Client-server communication must assume the client can be modified or impersonated.

### Assets Folder

The `assets` folder can store any type of file, such as HTML, fonts, mp3, text, and image files. The importance of this directory and its contents depends on what files are stored and how they are used.

### Lib Folder

This directory stores libraries and precompiled code, commonly found in subdirectories representing different CPU types and instruction sets known as [Application Binary Interfaces](https://developer.android.com/ndk/guides/abis.html) (ABIs). Examples include x86, x86_64, and armeabi. The .so files in these subdirectories are libraries created by the developer or from third parties. If an attacker modifies or replaces these files, it could result in arbitrary code execution.

### META-INF Folder

This directory contains files related to the integrity and authenticity of the application:

- **MANIFEST.MF**: A listing of all resource files and their SHA1 hashes.
- **CERT.RSA**: The developer’s signing certificate.
- **CERT.SF**: A list of resources and their hashes, corresponding to the MANIFEST.MF.

### Res Folder

This directory contains resources such as images that are not compiled into the `resource.arsc` file. Generally, these files are less impactful from a security perspective.

### Other Files

There can be other types of files and directories present for various reasons, including:

- App-specific customization and resource directories.
- Third-party libraries.
- HTML template files used in WebViews.

When auditing an app's source code, it's essential to review all files to determine their impact on the application's security.

## Code Signing

As mentioned previously, following the build process, `.apk` files need to be signed.
Andriod devices will not run uinsigned `.apk` files and whether you are building for testing or deployment, the process only varies by which keys are used to sign.

.
.
.
.
.
.
_In progress..._

## Let's Connect

I welcome your insights, feedback, and opportunities for collaboration. Together, we can make the digital world safer, one challenge at a time.

- **LinkedIn**: (https://www.linkedin.com/in/aashwadhaama/)

I look forward to connecting with fellow cybersecurity enthusiasts and professionals to share knowledge and work together towards a more secure digital environment.
