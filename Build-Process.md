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

As mentioned previously, following the build process, `.apk` files need to be signed. Android devices will not run unsigned `.apk` files, and whether you are building for testing or deployment, the process only varies by which keys are used to sign.

Public-key cryptography uses two keys: a private key and a public key. These keys are complementary, meaning one key can decrypt data encrypted by the other.

It is crucial that the private key remains secret within an organization, while the public key is intended to be shared.

The fact that one key can decrypt data encrypted by the other is used to verify the authenticity of an encrypted string. This is fundamental to how code signing works.

The public key is often included in a type of digital file known as an x.509 certificate. This certificate can be used to verify the identity of an entity.

The identity is established because the certificate includes information about the organization it belongs to, such as the name of the organization.

To understand how this is ensured, we first need to understand how cryptographic hashes work.

A cryptographic hash function is a special type of hash function where the output is theoretically unique for all inputs. If a single bit is changed in the input, the cryptographic hashing function will produce a completely different output. This fact is often used to verify that something has not been modified or to confirm that two inputs are identical.

Applying this notion that input will always produce the same, unique output, a certificate can be hashed to produce a value.

If this unique value is then encrypted with the private key or "signed," a person wanting to verify the certificate's authenticity can decrypt the encrypted hash using the public key and verify the value against their own hash of the certificate.

If the two hash values match, it confirms that the certificate is legitimate and has not been modified; otherwise, something has gone wrong.

Android apps are cryptographically signed in a similar fashion using a private key known only to the application developer. This process provides several key security-related features by:

- Validating the identity of the author
- Ensuring the code itself has not been modified after compiling

There have been several Android vulnerabilities identified related to the implementation of this protection.

Unlike other uses of public-key cryptography, there is no need to use CA (Certificate Authority) issued certificates. Self-signed certificates are just fine and no less secure.

Additionally, for Android, the subject name field on a certificate, which usually stores the entity's name, is not validated as part of the identity, nor is the expiration date considered. Android simply uses the certificate as a binary blob.

In part, the need for some of these features is diminished by the way the deployment of applications is accomplished.

The Android code signing process uses public-key cryptography with `x.509` certificates, similar to Java's jar signing process.

The actual process of signing the APK is built into the Android Studio IDE and largely abstracted from the developer.

Despite this fact, it is important to understand the details to know what opportunities exist for abuse and the exact extent of the security it provides.

For example, let's directly use `jarsigner`, which can accomplish the same tasks outside of the IDE. The tool is located in the bin folder of the **Java JDK install**.

Before signing the app, a private key needs to be generated using the `keytool`.

```
keytool -genkey -v -keystore foo.keystore -alias myalias -keyalg RSA -keysize 2048 -validity 10000
```
This example prompts for passwords for the keystore and key and to provide the Distinguished Name fields for the key. It then generates the keystore as a file called `foo.keystore`.

The keystore contains a single key, valid for 10,000 days. The alias is a name that can be used later when signing the app.

The `jarsigner` tool, which can be used to sign or verify a signature, is the same tool used for a traditional Java JAR file.

For example:

```
jarsigner -sigalg SHA1withRSA -digestalg SHA1 -keystore foo.keystore test.apk myalias
```

- **sigalg** specifies the signature algorithm used.
- **digestalg** specifies the digest algorithm used when digesting the file entries.
- **keystore** specifies a keystore file where the certificate files are stored and the file name to sign.
- **myalias** represents the alias in the keystore file.

To inspect the signing status of an existing APK:

```
jarsigner -verify -verbose -certs com.foo.android.activity.apk
```

Taking a look at an example of the `MANIFEST.MF` file, you can see a list of hashes and files:

```
Manifest-Version: 1.0
Created-By: 1.0 (Android)
Name: res/drawable-hdpi/ab_bg.9.png
SHA1-Digest: ihAK6Ph6K890IxHTZTDyU9UyYUc=
Name: res/drawable-xxhdpi/g7/png
SHA1-Digest: /zW+RqFe/jC0qd4/2NeALMCPTUU=
...
```

File hashes can be verified to be correct by using the openssl tool. If the hash matches the hash of the original file, we can be sure it has not been modified.

```
openssl sha1 -binary res/drawable-hdpi/ab_bg.9.png | openssl base64
```

The following command can be run to convert the format from DER to PEM and output a file named `cert.pem`:

```
openssl pkcs7 -inform DER -print_certs -out cert.pem -in CERT.RSA
```

To see the details for the public key in the certificate, run the following:

```
openssl x509 -in cert.pem -noout -text
```

### The Key is the Key

It is extremely important for organizations to protect the private key used to sign their production applications.

If it is ever compromised, there is no way for them to recover from this and continue to have their app users receive updates.

If a key is compromised, the organization will have to sign new versions with a different key, and the Google Play Store will treat it as a completely different app since this is how they identify the organization who signs the app.

Additionally, compromise could theoretically allow an attacker to publish malicious apps that would be trusted as if they were signed by the organization.

This is particularly damaging if the organization has multiple apps that rely on the signing to verify each other's identities.

### Signing Modes

Android Studio has two signing modes:

- **Debug**: Used for testing purposes and is never used to sign applications published to the public. This allows running apps directly connected via USB or on emulators.

- **Release**: Used for apps that are bound for consumer devices.

### APK Alignment

An APK must be aligned after it is signed. This is done using the zipalign tool located in the following directory:

```
<sdk install_location>/sdk/build-tools/<version>/
```
The purpose here is to improve RAM utilization when running the application.

```
zipalign -v 4 yourproject_unaligned.apk yourproject.apk
```

## Let's Connect

I welcome your insights, feedback, and opportunities for collaboration. Together, we can make the digital world safer, one challenge at a time.

- **LinkedIn**: (https://www.linkedin.com/in/aashwadhaama/)

I look forward to connecting with fellow cybersecurity enthusiasts and professionals to share knowledge and work together towards a more secure digital environment.
