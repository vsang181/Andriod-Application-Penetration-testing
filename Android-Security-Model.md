# Android Security Model 

## Application Sandbox 

The Android platform uses Linux user-based protection to identify and isolate app resources. This ensures that apps are isolated from each other and protects both apps and the system from malicious activities. Each Android application is assigned a unique user ID (UID) and runs in its own process. 

Android sets up a kernel-level Application Sandbox using the UID. The kernel enforces security between apps and the system through standard Linux facilities such as user and group IDs. By default, apps cannot interact with each other and have limited access to the OS. If app A tries to read app B's data or dial the phone without permission, it is prevented from doing so because it lacks the necessary privileges. The sandbox, based on UNIX-style user separation of processes and file permissions, is simple and auditable. 

The Application Sandbox is in the kernel, extending this security model to both native code and OS applications. All software above the kernel, such as OS libraries, application framework, application runtime, and all applications, run within the Application Sandbox. On Android, there are no restrictions on how an application can be written; native code is sandboxed just like interpreted code. 

### Protections 

To break out of the Application Sandbox on a properly configured device, one must compromise the Linux kernel's security. However, defense-in-depth is essential as individual protections are not invulnerable. 

Android uses several protections to enforce the Application Sandbox: 

- Android 5.0: SELinux provided mandatory access control (MAC) separation between the system and apps. Third-party apps ran within the same SELinux context, so inter-app isolation was primarily enforced by UID DAC. 

- Android 6.0: The SELinux sandbox was extended to isolate apps across the per-physical-user boundary. Safer defaults for application data were set; for apps with `targetSdkVersion >= 24`, default DAC permissions on an app's home dir changed from 751 to 700. 

- Android 8.0: All apps were set to run with a seccomp-bpf filter, limiting the syscalls apps could use and strengthening the app/kernel boundary. 

- Android 9: Non-privileged apps with `targetSdkVersion >= 28` must run in individual SELinux sandboxes, providing MAC on a per-app basis. 

- Android 10: Apps have a limited raw view of the filesystem, with no direct access to paths like `/sdcard/DCIM`. However, apps retain full raw access to their package-specific paths. 

### Guidelines for Sharing Files 

Setting app data as world-accessible is poor security practice and is disallowed for apps with targetSdkVersion >= 28 in Android 9 and higher. Instead, use these guidelines: 

- **Content Provider**: If your app needs to share files with another app, use a content provider. Content providers share data with the proper granularity and without the downsides of world-accessible UNIX permissions. 

- **MediaStore Class**: For files that should be accessible to the world (photos, videos, audio), use the MediaStore class. 

The Storage runtime permission controls access to strongly-typed collections through MediaStore. For accessing weakly typed files such as PDFs, use intents like the `ACTION_OPEN_DOCUMENT` intent. 

To enable Android 10 behavior, use the `requestLegacyExternalStorage` manifest attribute and follow App permissions best practices: 

- Default value is true for apps targeting Android 9 (and lower). 

- Default value is false for apps targeting Android 10. To temporarily opt out of the filtered storage view in apps targeting Android 10, set the manifest flag’s value to true. 

Using restricted permissions, the installer whitelists apps permitted for non-sandboxed storage. Non-whitelisted apps are sandboxed. 

## OMAPI Vendor Stable Interface 

### Introduction 

Open Mobile API (OMAPI) is a standard API used to communicate with a device's Secure Element. Prior to Android 13, only applications and framework modules could access this interface. Now, by converting it to a vendor stable interface, HAL modules can also communicate with Secure Elements through the OMAPI service. 

A new access entry for OMAPI was added for HAL modules without modifying any existing APIs. Existing applications and framework modules using this interface require no changes. 

As part of the Android Ready SE program, core Android security features like Keymaster, Keymint, Identity Credentials, and Remote Key Provisioning are made available on Secure Elements. Enabling these requires HALs (vendor components) of these features to communicate with the Secure Element via the OMAPI vendor stable interface. 

### Design Architecture

![omapi-design-architecture](https://github.com/vsang181/Andriod-Application-Penetration-testing/assets/28651683/86e78c4b-790a-4335-a120-46f86c460760)

OEMs integrating a Secure Element and Android Ready SE features into their devices need to enable this interface as it is disabled by default. Before this update, Secure Element access rules were defined by package name or its signature hashes (device application reference) and AID (SE application reference). HAL modules didn’t have unique identifiers like package names or signature certificates. Now in Android 13, the OMAPI Vendor Stable Service allows HAL modules to access the Secure Element. SE vendors can define a unique identifier UUID of 16 bytes. To apply this access rule to HAL modules, SE vendors are required to map this 16-byte unique identifier UUID to HAL module UID in their vendor UUID mapping config XML.

The OMAPI Vendor Stable Service pads the UUID with FF if necessary to make it 20 bytes, as per section 6.1, DeviceAppID-REF-DO page 66, and defines access rules in secure elements using this 20-byte UUID as the device application reference.

The vendor UUID mapping file name is formed with the predefined prefix `hal_uuid_map_` and appended with the value of the system property `ro.boot.product.hardware.sku`:

```
hal_uuid_map_value_of_ro.boot.product.hardware.sku.xml
```

The OMAPI Vendor Stable Service searches for this file under `/odm/etc/`, `/vendor/etc/`, and `/etc/` folders. A detailed description of the vendor UUID mapping configuration file is available [here](https://cs.android.com/android/platform/superproject/+/main:packages/apps/SecureElement/etc/hal_uuid_map_config.xml).

### Implementation

The following changes are required to enable the OMAPI Vendor Stable Service feature on a target build.

#### SecureElement

1. Enable the Service Flag

Enable the service flag `secure_element_vintf_enabled` using a resource overlay under device-specific folders:

```
<bool name="secure_element_vintf_enabled">true</bool>
```

2. Define the UID and UUID Mapping XML

Define the UID and UUID mapping XML for your service:

```
<ref_do>
    <uuid_ref_do>
        <uids>
            <uid>0</uid>
        </uids>
        <uuid>9f36407ead0639fc966f14dde7970f68</uuid>
    </uuid_ref_do>
    <uuid_ref_do>
        <uids>
            <uid>1096</uid>
            <uid>1097</uid>
        </uids>
        <uuid>a9b7ba70783b317e9998dc4dd82eb3c5</uuid>
    </uuid_ref_do>
</ref_do>
```

Provision the Secure Element ARs for the HAL service using UUIDs as device application references. Add a mapping entry in the mapping config where you can map this UUID to HAL module UID(s). This mapping allows HAL modules to access the Secure Element. OMAPI VTS tests can be used as reference implementations for enabling the OMAPI Vendor Stable Service in HAL modules.

3. Update HAL Module sepolicy

Add a sepolicy rule for the HAL module to allow their domain to access the OMAPI vendor stable service:

```
allow hal_module_label secure_element_service:service_manager find;
```

4. Connect to OMAPI Vendor Stable Service

From HAL modules, use the OMAPI vendor service label `android.se.omapi.ISecureElementService/default` to connect to the service.

### Validation

Validate that the OMAPI Vendor Stable Service has been successfully implemented by running OMAPI VTS tests:

```
run vts -m VtsHalOmapiSeServiceV1_TargetTest
run vts -m VtsHalOmapiSeAccessControlTestCases
```

## Let's Connect

I welcome your insights, feedback, and opportunities for collaboration. Together, we can make the digital world safer, one challenge at a time.

- **LinkedIn**: (https://www.linkedin.com/in/aashwadhaama/)

I look forward to connecting with fellow cybersecurity enthusiasts and professionals to share knowledge and work together towards a more secure digital environment.
