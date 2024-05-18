# Android Architecture 

![image](https://github.com/vsang181/Andriod-Application-Penetration-testing/assets/28651683/12424687-99e7-422b-a8cd-9b2fd845cec0)

As shown in the image there are several general layers in the Android operating system, each of which has a specialized set of functionalities. 

### System Apps 

Android includes core apps for email, SMS, calendar, browsing, contacts, and more. These apps have no special status over user-installed apps; third-party apps can become default browsers, SMS messengers, or keyboards, except for system settings. System apps serve both as user apps and provide key functionalities accessible to developers. For instance, your app can use the installed SMS app to send messages without building that feature yourself. 

### Java APi Framework 

Android's entire feature set is available through Java APIs, simplifying app development by reusing core system components and services. Key components include: 

- A rich, extensible view system for building UIs with lists, grids, text boxes, buttons, and an embeddable web browser. 

- A resource manager for accessing non-code resources like localized strings, graphics, and layout files. 

- A notification manager for displaying custom alerts in the status bar. 

- An activity manager for handling app lifecycles and navigation. 

- Content providers for accessing and sharing data between apps, like the Contacts app. 

Developers have full access to the same framework APIs as system apps. 

## Native C/C++ libraries 

Core Android components like ART and HAL are built with native C/C++ libraries. The platform provides Java APIs to access these libraries' functionality. For instance, you can use the Java OpenGL API to incorporate 2D and 3D graphics in your app. 

For apps requiring C/C++ code, the Android NDK allows direct access to these native libraries. 

### Android Runtime 

For Android 5.0 (API level 21) and above, each app runs in its own process with its own instance of Android Runtime (ART). ART runs multiple virtual machines on low-memory devices using Dalvik Executable (DEX) files, a bytecode format optimized for minimal memory footprint. Java sources are compiled into DEX bytecode using tools like d8. 

Key features of ART include: 

- Ahead-of-time (AOT) and just-in-time (JIT) compilation 

- Optimized garbage collection (GC) 

- Conversion of DEX files to compact machine code on Android 9 (API level 28) and higher 

- Enhanced debugging support, including a sampling profiler, detailed diagnostic exceptions, crash reporting, and watchpoints 

Before Android 5.0, Dalvik was the runtime. Apps running well on ART are compatible with Dalvik, but the reverse might not hold. Android also provides core runtime libraries offering most Java language functionality, including some Java 8 features. 

### Hardware Abstraction Layer (HAL) 

The Hardware Abstraction Layer (HAL) provides standard interfaces to expose hardware capabilities to the Java API framework. It consists of multiple library modules, each implementing an interface for a specific hardware component, like the camera or Bluetooth. When a framework API calls to access hardware, the Android system loads the corresponding library module. 

## Linux Kernel 

The Linux kernel forms the foundation of the Android platform. The Android Runtime (ART) relies on it for threading and low-level memory management. Using the Linux kernel enables Android to leverage key security features and allows device manufacturers to develop hardware drivers for a familiar kernel. 

The kernel used in Android is based on the Linux 2.6 kernel. If like to learn more about it, can find more details [here](https://en.wikipedia.org/wiki/Linux_kernel). 

## Android Runtime and Dalvik

![image](https://github.com/vsang181/Andriod-Application-Penetration-testing/assets/28651683/5cf9827a-15e1-48a9-bf19-bc75f8bcd6c2)

The Android Runtime (ART) is the managed runtime used by apps and some system services on Android. Both ART and its predecessor, Dalvik, were specifically created for the Android project. ART executes the Dalvik Executable (DEX) format and DEX bytecode specification. 

ART and Dalvik are compatible runtimes that run DEX bytecode, so apps developed for Dalvik should work with ART. However, some techniques that work on Dalvik do not work on ART. For more information on these issues, see [Verifying app behavior on the Android runtime (ART)](https://developer.android.com/guide/practices/verifying-apps-art.html). 

### ART Features 

#### Ahead-of-time (AOT) Compilation 

ART introduces ahead-of-time (AOT) compilation, which can improve app performance. It also has tighter install-time verification than Dalvik. 

At install time, ART compiles apps using the on-device dex2oat tool. This tool accepts DEX files as input and generates a compiled app executable for the target device. It should compile all valid DEX files without difficulty, but some post-processing tools produce invalid files that may be tolerated by Dalvik but cannot be compiled by ART. For more information, see [Addressing Garbage Collection Issues](https://developer.android.com/guide/practices/verifying-apps-art.html#GC_Migration). 

#### Improved Garbage Collection 

Garbage collection (GC) is resource-intensive and can impair app performance, resulting in choppy display, poor UI responsiveness, and other issues. ART improves garbage collection in several ways: 

- Mostly concurrent design with a single GC pause 

- Concurrent copying to reduce background memory usage and fragmentation 

- The length of the GC pause is independent of the heap size 

- Collector with lower total GC time for cleaning up recently-allocated, short-lived objects 

- Improved garbage collection ergonomics, making concurrent garbage collections more timely, making `GC_FOR_ALLOC` events extremely rare in typical use cases 

#### Development and Debugging Improvements 

ART offers several features to improve app development and debugging. 

#### Support for Sampling Profiler 

Historically, developers used the Traceview tool (designed for tracing app execution) as a profiler. While Traceview provided useful information, its results on Dalvik were skewed by the per-method-call overhead, and its use noticeably affected runtime performance. 

ART adds support for a dedicated sampling profiler without these limitations, providing a more accurate view of app execution without significant slowdown. Sampling support was added to Traceview for Dalvik in the KitKat release. 

#### Support for More Debugging Features 

ART supports new debugging options, particularly related to monitor and garbage collection functionality. For example, you can: 

- See what locks are held in stack traces and jump to the thread holding a lock. 

- Ask how many live instances exist of a given class, view the instances, and see what references keep an object alive. 

- Filter events (like breakpoints) for a specific instance. 

- See the value returned by a method when it exits (using “method-exit” events). 

- Set field watchpoints to suspend program execution when a specific field is accessed and/or modified. 

#### Improved Diagnostic Detail in Exceptions and Crash Reports 

ART provides detailed context when runtime exceptions occur, offering expanded exception detail for `java.lang.ClassCastException`, `java.lang.ClassNotFoundException`, and `java.lang.NullPointerException`.  

> For example, `java.lang.NullPointerException` now shows what the app was trying to do with the null pointer, such as the field it was trying to write to or the method it was trying to call.  

Examples include: 

``` 

java.lang.NullPointerException: Attempt to write to field 'int  

android.accessibilityservice.AccessibilityServiceInfo.flags' on a null object  

reference 

``` 

& 

``` 

java.lang.NullPointerException: Attempt to invoke virtual method  

'java.lang.String java.lang.Object.toString()' on a null object reference 

``` 

ART also improves context information in app native crash reports by including both Java and native stack information. 

## Let's Connect

I welcome your insights, feedback, and opportunities for collaboration. Together, we can make the digital world safer, one challenge at a time.

- **LinkedIn**: (https://www.linkedin.com/in/aashwadhaama/)

I look forward to connecting with fellow cybersecurity enthusiasts and professionals to share knowledge and work together towards a more secure digital environment.
