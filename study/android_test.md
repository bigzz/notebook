# Android Testing Tools

Testing is a critical software development activity because it helps you improve the quality of your apps, ensure better user satisfaction, and reduce overall development time spent on fixing defects.

The following sections describe tools that help you test your mobile apps for the Android platform.

**[Android Testing Support Library](http://developer.android.com/tools/testing-support-library/index.html)**

This library provides a set of APIs that allow you to quickly build and run test code for your apps, including JUnit 4 and functional user interface (UI) tests. The Android Testing Support Library includes the following test automation tools:

* **AndroidJUnitRunner**: JUnit 4-compatible test runner for Android

* 
**Espresso**: UI testing framework; suitable for functional UI testing within an app

* 
**UI Automator**: UI testing framework; suitable for cross-app functional UI testing across system and installed apps


**[Monkey](http://developer.android.com/tools/help/monkey.html)**

This tool runs on your emulator or device and generates pseudo-random streams of user events such as clicks, touches, or gestures, as well as a number of system-level events. You can use the Monkey tool to stress-test applications that you are developing, in a random yet repeatable manner.

**[monkeyrunner](http://developer.android.com/tools/help/monkeyrunner_concepts.html)**

This testing system provides an API for writing programs that control an Android device or emulator from outside of Android code.