# AOSP Automotive SystemUI Guide

A practical guide for understanding, customizing, building, and debugging **SystemUI** in **AOSP Automotive / Android Automotive OS (AAOS)**.

## Who This Guide Is For

This README is for Android Automotive engineers who need to understand how SystemUI works, how `CarSystemUI` differs from phone/tablet SystemUI, and how to safely customize system bars, overlays, vehicle-aware UI, and app-state-driven behavior.

## Table of Contents

- [Who This Guide Is For](#who-this-guide-is-for)
- [1. What is SystemUI?](#1-what-is-systemui)
- [2. SystemUI in Normal Android vs Android Automotive](#2-systemui-in-normal-android-vs-android-automotive)
- [3. Where SystemUI Lives Inside AOSP](#3-where-systemui-lives-inside-aosp)
- [4. Main Responsibilities of SystemUI](#4-main-responsibilities-of-systemui)
  - [4.1 Draw System Bars](#41-draw-system-bars)
  - [4.2 Listen to System State](#42-listen-to-system-state)
  - [4.3 Manage Overlays](#43-manage-overlays)
  - [4.4 Control Navigation](#44-control-navigation)
  - [4.5 Handle Notifications](#45-handle-notifications)
  - [4.6 Multi-Display Support](#46-multi-display-support)
- [5. Important SystemUI Components](#5-important-systemui-components)
- [6. How SystemUI Starts When Android Boots](#6-how-systemui-starts-when-android-boots)
- [7. SystemUI Architecture](#7-systemui-architecture)
- [8. Automotive SystemUI / CarSystemUI](#8-automotive-systemui--carsystemui)
- [9. How to Customize Existing SystemUI](#9-how-to-customize-existing-systemui)
  - [9.1 Change Layouts](#91-change-layouts)
  - [9.2 Change Drawables](#92-change-drawables)
  - [9.3 Change Dimensions](#93-change-dimensions)
  - [9.4 Change Colors](#94-change-colors)
  - [9.5 Change Behavior in Controllers](#95-change-behavior-in-controllers)
- [10. How to Create Your Own SystemUI](#10-how-to-create-your-own-systemui)
  - [10.1 Option A: Customize Existing SystemUI](#101-option-a-customize-existing-systemui)
  - [10.2 Option B: Create a New SystemUI APK from Scratch](#102-option-b-create-a-new-systemui-apk-from-scratch)
  - [10.3 Step 1: Create Module Structure](#103-step-1-create-module-structure)
  - [10.4 Step 2: AndroidManifest.xml](#104-step-2-androidmanifestxml)
  - [10.5 Step 3: Create SystemUI Service](#105-step-3-create-systemui-service)
  - [10.6 Step 4: Create System Bar Controller](#106-step-4-create-system-bar-controller)
  - [10.7 Step 5: Create Custom View](#107-step-5-create-custom-view)
  - [10.8 Step 6: Layout XML](#108-step-6-layout-xml)
  - [10.9 Step 7: Android.bp](#109-step-7-androidbp)
  - [10.10 Step 8: Add to Product Makefile](#1010-step-8-add-to-product-makefile)
  - [10.11 Step 9: Install as priv-app](#1011-step-9-install-as-priv-app)
- [11. Build and Push SystemUI APK](#11-build-and-push-systemui-apk)
- [12. Common Automotive SystemUI Use Cases](#12-common-automotive-systemui-use-cases)
  - [12.1 Add New Button to Side Bar](#121-add-new-button-to-side-bar)
  - [12.2 Detect Which App Is on Top](#122-detect-which-app-is-on-top)
  - [12.3 Change UI Based on Night Mode](#123-change-ui-based-on-night-mode)
  - [12.4 Show Glow When Map Is Visible](#124-show-glow-when-map-is-visible)
- [13. Debugging SystemUI](#13-debugging-systemui)
  - [13.1 Check Logs](#131-check-logs)
  - [13.2 Check Crash Logs](#132-check-crash-logs)
  - [13.3 Restart SystemUI Manually](#133-restart-systemui-manually)
  - [13.4 Check Installed APK Path](#134-check-installed-apk-path)
  - [13.5 Check Permissions](#135-check-permissions)
  - [13.6 Check Windows](#136-check-windows)
  - [13.7 Check Top Activity](#137-check-top-activity)
- [14. Best Practices](#14-best-practices)
  - [14.1 Do Not Put Heavy Logic in Views](#141-do-not-put-heavy-logic-in-views)
  - [14.2 Keep SystemUI Lightweight](#142-keep-systemui-lightweight)
  - [14.3 Be Careful with Animations](#143-be-careful-with-animations)
  - [14.4 Always Test on Real Hardware](#144-always-test-on-real-hardware)
  - [14.5 Keep a Backup Before Pushing APK](#145-keep-a-backup-before-pushing-apk)
  - [14.6 Add Safe Logs](#146-add-safe-logs)
- [15. Final Mental Model](#15-final-mental-model)
- [16. Recommended Practical Exercise](#16-recommended-practical-exercise)

---

## 1. What is SystemUI?

**SystemUI** is a privileged Android system application responsible for drawing and controlling many parts of the system interface that are not part of normal apps.

- It is not your launcher.
- It is not Settings.
- It is not an app that users normally open.

It is a **core system app** that runs all the time and controls things like:

* Status bar
* Navigation bar
* Notification shade
* Quick settings
* Lock screen UI
* Volume dialog
* Power menu
* Privacy indicators
* System overlays
* Heads-up notifications
* Automotive system bars
* HVAC/system controls in some car builds
* Edge panels / contextual controls in car UI

In simple words:

> SystemUI is the Android system app that draws and manages the UI around apps.

For example, when you open YouTube, Chrome, Maps, or any app, the app content is drawn by that app. But the top bar, bottom bar, navigation buttons, notification panel, and some system-level overlays are controlled by **SystemUI**.

---

## 2. SystemUI in Normal Android vs Android Automotive

In a normal phone/tablet Android system, SystemUI usually handles:

```text
Top status bar
Bottom navigation bar
Notification shade
Quick settings
Lock screen
Volume panel
Power menu
```

Example:

```text
+--------------------------------+
| Status Bar: Wi-Fi Battery Time |
+--------------------------------+
|                                |
|           App Content          |
|                                |
|                                |
+--------------------------------+
| Back | Home | Recent           |
+--------------------------------+
```

In **Android Automotive**, SystemUI is different because the car has a very different UX.

Automotive may have:

* Left system bar
* Right system bar
* Bottom system bar
* Top system bar
* Passenger display
* Driver display
* Center display
* HVAC controls
* App launcher button
* Home button
* Notification center
* Assistant button
* Volume/climate overlays
* Rear object detection UI
* Map overlays
* Contextual home screen areas
* Multiple displays
* Safety restrictions
* Car-specific user switching

Example Automotive layout:

```text
+--------------------------------------------------+
|                  Top System Bar                  |
+------+-----------------------------------+-------+
|Left  |                                   |Right  |
|Bar   |          App / Map / Home         |Bar    |
|      |                                   |       |
+------+-----------------------------------+-------+
|                Bottom System Bar                 |
+--------------------------------------------------+
```

In many AOSP Automotive projects, SystemUI is customized heavily because OEMs want a unique car experience.

That is why you may see apps or modules like:

```text
SystemUI
CarSystemUI
CustomCarSystemUI
CarLauncher
CarSystemBar
```

In a real Automotive IVI project, you will often deal with a customized version of **CarSystemUI**.

---

## 3. Where SystemUI Lives Inside AOSP

In AOSP, the common SystemUI source code is usually here:

```bash
frameworks/base/packages/SystemUI/
```

Important folders:

```bash
frameworks/base/packages/SystemUI/src/
frameworks/base/packages/SystemUI/res/
frameworks/base/packages/SystemUI/Android.bp
frameworks/base/packages/SystemUI/AndroidManifest.xml
```

For Automotive, you may also find car-specific SystemUI code under:

```bash
packages/apps/Car/SystemUI/
```

or sometimes:

```bash
packages/apps/Car/CarSystemUI/
```

Depending on the project/vendor customization, it may also be under vendor-specific paths, for example:

```bash
vendor/<company>/apps/CarSystemUI/
packages/apps/Car/SystemUI/
device/<vendor>/<product>/...
```

In your project, based on your previous work, you have paths like:

```bash
CustomCarSystemUI
CarSystemBarLeftView
CarSystemBarRightView
CarSystemBarView.java
```

So your actual SystemUI is probably a customized Automotive SystemUI APK.

---

## 4. Main Responsibilities of SystemUI

SystemUI has many responsibilities. The most important ones are:

### 4.1 Draw System Bars

SystemUI draws bars around apps.

Examples:

```text
Status bar
Navigation bar
Left car bar
Right car bar
Bottom car bar
```

In Automotive, this could mean:

```text
Home button
App launcher
HVAC shortcut
Volume icon
Profile icon
Notifications icon
Map controls
Clock
Signal status
```

---

### 4.2 Listen to System State

SystemUI listens to Android system events, such as:

```text
Battery changed
Wi-Fi changed
Bluetooth changed
User switched
Screen locked
App changed
Display changed
Navigation mode changed
Theme changed
Car property changed
```

In Automotive, it may also listen to:

```text
Gear state
Driving state
Night mode
Display theme
HVAC state
Vehicle speed
Seat belt state
Map visibility
VHAL signals
```

---

### 4.3 Manage Overlays

SystemUI can show UI on top of other apps.

Examples:

```text
Volume dialog
Toast-like system overlay
Privacy chip
Assistant overlay
Notification heads-up
Power menu
Car HVAC overlay
```

---

### 4.4 Control Navigation

SystemUI often owns navigation buttons:

```text
Back
Home
Recents
App launcher
Voice assistant
```

In Automotive, navigation is usually custom.

Example:

```text
Left bar:
- Home
- Apps
- Navigation
- Phone
- Media
```

---

### 4.5 Handle Notifications

On phones, SystemUI owns the notification shade.

In Automotive, notifications may appear in:

```text
Notification center
Heads-up card
Side panel
Cluster-safe notification UI
```

---

### 4.6 Multi-Display Support

Automotive systems may have multiple displays:

```text
Center display
Passenger display
Rear display
Instrument cluster
Secondary display
Primary display
```

SystemUI may need to know:

```text
Which display is active?
Which app is on top?
Which display should show the bar?
Should each display have an independent home/context state?
```

This is very important in Automotive AOSP.

---

## 5. Important SystemUI Components

SystemUI is not one simple Activity. It is a large system app made of services, controllers, views, and managers.

Common pieces include:

```text
SystemUIApplication
SystemUIService
SystemUIInitializer
StatusBar
NavigationBar
CommandQueue
NotificationShadeWindowController
KeyguardViewMediator
Dependency injection classes
Controllers
Views
Managers
```

In Automotive, you may see classes like:

```text
CarSystemUIFactory
CarSystemBar
CarSystemBarController
CarSystemBarView
CarNavigationBar
CarStatusBar
CarNotificationView
FullscreenUserSwitcher
HVAC controllers
```

A simplified structure looks like this:

```text
SystemUI APK
│
├── Application class
│
├── Service
│
├── Controllers
│   ├── StatusBarController
│   ├── NavigationBarController
│   ├── NotificationController
│   ├── CarSystemBarController
│
├── Views
│   ├── status_bar.xml
│   ├── navigation_bar.xml
│   ├── car_left_system_bar.xml
│   ├── car_right_system_bar.xml
│
├── Resources
│   ├── drawables
│   ├── colors
│   ├── dimens
│   ├── styles
│
└── Manifest
```

---

## 6. How SystemUI Starts When Android Boots

SystemUI is started by the Android framework during boot.

The high-level flow is:

```text
Android boots
↓
SystemServer starts
↓
System services start
↓
ActivityManagerService becomes ready
↓
SystemUIService is started
↓
SystemUI initializes components
↓
Status bar / navigation bar / car bars are attached to screen
```

Usually SystemUI is started from framework code using something like:

```java
startServiceAsUser(
    new Intent().setComponent(SystemUIServiceComponent),
    UserHandle.SYSTEM
);
```

The SystemUI APK contains a service like:

```xml
<service
    android:name=".SystemUIService"
    android:exported="true" />
```

The important thing to understand:

> SystemUI does not start because the user opens it.
> Android itself starts SystemUI as part of the system boot process.

That is why if SystemUI crashes, Android usually tries to restart it.

You may see logs like:

```text
SystemUIService started
SystemUI crash
Restarting SystemUI
```

---

## 7. SystemUI Architecture

SystemUI usually follows this kind of architecture:

```text
View
↓
Controller
↓
Service / Manager
↓
Android framework / Car service
```

Example:

```text
CarSystemBarView
↓
CarSystemBarController
↓
ActivityTaskManager / CarPropertyManager
↓
System state / Vehicle state
```

Another example:

```text
Battery icon view
↓
BatteryController
↓
BatteryManager
↓
Android battery service
```

For Automotive:

```text
Theme icon / night mode UI
↓
ThemeController
↓
CarPropertyManager
↓
VHAL signal
```

Or:

```text
Map edge glow view
↓
TaskStackChangeListener / ActivityTaskManager
↓
Detect top activity
↓
Show or hide map-specific background
```

This architecture is important because you should not put all logic inside a View.

Bad approach:

```java
public class CarSystemBarView extends LinearLayout {
    // too much logic here
    // reading car property
    // checking top activity
    // changing theme
    // controlling buttons
}
```

Better approach:

```text
CarSystemBarView
    only displays UI

CarSystemBarController
    controls behavior

CarPropertyRepository / Manager
    talks to car APIs

TaskMonitor
    monitors top activity
```

---

## 8. Automotive SystemUI / CarSystemUI

In Automotive, SystemUI becomes much more important because it is part of the car HMI.

A typical Automotive SystemUI may control:

```text
Left system bar
Right system bar
Bottom bar
Top bar
Notification center
HVAC panel
Volume UI
User switcher
App grid button
Home button
Assistant button
Camera overlay
Parking overlay
Map contextual UI
```

A car SystemUI may have layout files like:

```bash
res/layout/car_system_bar.xml
res/layout/car_left_system_bar.xml
res/layout/car_right_system_bar.xml
res/layout/car_top_system_bar.xml
res/layout/car_bottom_system_bar.xml
```

And classes like:

```java
CarSystemBar.java
CarSystemBarView.java
CarSystemBarController.java
CarNavigationBarController.java
```

The idea is:

```text
The layout defines the UI.
The View holds references to buttons/icons.
The Controller decides what should happen.
The system services provide state.
```

For example, when the user clicks the Home button:

```java
homeButton.setOnClickListener(v -> {
    activityStarter.startHomeActivity();
});
```

Or when a map app is on top:

```java
if (isNativeMapOnTop) {
    showMapEdgeGlow();
} else {
    hideMapEdgeGlow();
}
```

---

## 9. How to Customize Existing SystemUI

Most OEM work does **not** start by creating SystemUI from zero.

Usually you customize existing SystemUI.

Common customization areas:

### 9.1 Change Layouts

Example:

```bash
res/layout/car_left_system_bar.xml
```

You can add or remove buttons:

```xml
<ImageButton
    android:id="@+id/home_button"
    android:layout_width="64dp"
    android:layout_height="64dp"
    android:src="@drawable/ic_home" />
```

---

### 9.2 Change Drawables

Example:

```bash
res/drawable/system_bar_background.xml
res/drawable/ic_home.xml
res/drawable/map_edge_glow.xml
```

You can change:

```text
Icons
Backgrounds
Gradients
Glow effects
Masks
Radii
Borders
```

---

### 9.3 Change Dimensions

Example:

```bash
res/values/dimens.xml
```

```xml
<dimen name="car_system_bar_width">120dp</dimen>
<dimen name="car_system_bar_icon_size">48dp</dimen>
```

---

### 9.4 Change Colors

Example:

```bash
res/values/colors.xml
```

```xml
<color name="system_bar_background">#101010</color>
<color name="system_bar_icon_active">#FFFFFF</color>
<color name="system_bar_icon_inactive">#777777</color>
```

---

### 9.5 Change Behavior in Controllers

Example:

```java
private void updateMapState(boolean visible) {
    mLeftBarView.setMapMode(visible);
    mRightBarView.setMapMode(visible);
}
```

This is where you add logic like:

```text
If Native Map is visible, show glow.
If HVAC is open, hide contextual home.
If user switches display, update selected icon.
If night mode changes, change theme.
```

---

## 10. How to Create Your Own SystemUI

There are two meanings of “make my own SystemUI”.

### 10.1 Option A: Customize Existing SystemUI

This is the recommended way.

You keep the same package/service integration, then modify:

```text
Layouts
Drawables
Controllers
Buttons
Resources
Behavior
```

This is safer because Android already knows how to start your SystemUI.

---

### 10.2 Option B: Create a New SystemUI APK from Scratch

This is possible, but harder.

To make your own SystemUI, you need:

```text
Privileged app
Correct package
Correct permissions
SystemUI service
Correct manifest
Platform signing
Product makefile inclusion
Framework config pointing to your service
SELinux compatibility
Overlay permissions
Window permissions
Car service permissions
```

A basic custom SystemUI needs:

```text
AndroidManifest.xml
SystemUIApplication
SystemUIService
Main controller
System bar views
Layouts
Android.bp
priv-app installation
platform certificate
```

---

### 10.3 Step 1: Create Module Structure

Example:

```bash
packages/apps/MyCarSystemUI/
│
├── Android.bp
├── AndroidManifest.xml
├── src/com/example/mycarsystemui/
│   ├── MySystemUIApplication.java
│   ├── MySystemUIService.java
│   ├── MySystemBarController.java
│   └── MySystemBarView.java
│
└── res/
    ├── layout/
    │   └── my_system_bar.xml
    ├── drawable/
    ├── values/
    │   ├── colors.xml
    │   ├── dimens.xml
    │   └── strings.xml
```

---

### 10.4 Step 2: AndroidManifest.xml

Example:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.mycarsystemui">

    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
    <uses-permission android:name="android.permission.STATUS_BAR" />
    <uses-permission android:name="android.permission.MANAGE_ACTIVITY_TASKS" />
    <uses-permission android:name="android.permission.INTERACT_ACROSS_USERS" />
    <uses-permission android:name="android.permission.CONTROL_REMOTE_APP_TRANSITION_ANIMATIONS" />

    <application
        android:name=".MySystemUIApplication"
        android:persistent="true"
        android:theme="@style/MySystemUITheme"
        android:directBootAware="true">

        <service
            android:name=".MySystemUIService"
            android:exported="true"
            android:directBootAware="true" />

    </application>
</manifest>
```

Important:

```xml
android:persistent="true"
```

means the system treats it as an important always-running app.

But this only works properly for privileged/system apps.

---

### 10.5 Step 3: Create SystemUI Service

Example:

```java
package com.example.mycarsystemui;

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.util.Log;

public class MySystemUIService extends Service {

    private static final String TAG = "MySystemUIService";
    private MySystemBarController mSystemBarController;

    @Override
    public void onCreate() {
        super.onCreate();

        Log.d(TAG, "My SystemUI Service started");

        mSystemBarController = new MySystemBarController(this);
        mSystemBarController.start();
    }

    @Override
    public void onDestroy() {
        super.onDestroy();

        if (mSystemBarController != null) {
            mSystemBarController.destroy();
        }
    }

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
}
```

---

### 10.6 Step 4: Create System Bar Controller

```java
package com.example.mycarsystemui;

import android.content.Context;
import android.graphics.PixelFormat;
import android.view.Gravity;
import android.view.LayoutInflater;
import android.view.WindowManager;
import android.view.WindowManager.LayoutParams;
import android.util.Log;

public class MySystemBarController {

    private static final String TAG = "MySystemBarController";

    private final Context mContext;
    private final WindowManager mWindowManager;
    private MySystemBarView mBarView;

    public MySystemBarController(Context context) {
        mContext = context;
        mWindowManager = context.getSystemService(WindowManager.class);
    }

    public void start() {
        Log.d(TAG, "Starting custom system bar");

        mBarView = (MySystemBarView) LayoutInflater.from(mContext)
                .inflate(R.layout.my_system_bar, null);

        LayoutParams params = new LayoutParams(
                120,
                LayoutParams.MATCH_PARENT,
                LayoutParams.TYPE_APPLICATION_OVERLAY,
                LayoutParams.FLAG_NOT_FOCUSABLE
                        | LayoutParams.FLAG_NOT_TOUCH_MODAL
                        | LayoutParams.FLAG_LAYOUT_IN_SCREEN,
                PixelFormat.TRANSLUCENT
        );

        params.gravity = Gravity.START | Gravity.TOP;

        mWindowManager.addView(mBarView, params);
    }

    public void destroy() {
        if (mBarView != null) {
            mWindowManager.removeView(mBarView);
            mBarView = null;
        }
    }
}
```

But in real AOSP SystemUI, you usually do **not** use normal `TYPE_APPLICATION_OVERLAY`.

For true SystemUI, you may use system-level window types such as:

```java
TYPE_STATUS_BAR
TYPE_NAVIGATION_BAR
TYPE_NAVIGATION_BAR_PANEL
TYPE_STATUS_BAR_SUB_PANEL
```

Those require privileged permissions and platform integration.

For Automotive, your project may already have custom window handling inside `CarSystemUI`.

---

### 10.7 Step 5: Create Custom View

```java
package com.example.mycarsystemui;

import android.content.Context;
import android.util.AttributeSet;
import android.widget.LinearLayout;
import android.widget.ImageButton;
import android.content.Intent;

public class MySystemBarView extends LinearLayout {

    private ImageButton mHomeButton;

    public MySystemBarView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();

        mHomeButton = findViewById(R.id.home_button);

        mHomeButton.setOnClickListener(v -> goHome());
    }

    private void goHome() {
        Intent intent = new Intent(Intent.ACTION_MAIN);
        intent.addCategory(Intent.CATEGORY_HOME);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        getContext().startActivity(intent);
    }
}
```

---

### 10.8 Step 6: Layout XML

```xml
<com.example.mycarsystemui.MySystemBarView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/my_system_bar"
    android:layout_width="120dp"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="#111111"
    android:gravity="center">

    <ImageButton
        android:id="@+id/home_button"
        android:layout_width="72dp"
        android:layout_height="72dp"
        android:background="@android:color/transparent"
        android:src="@drawable/ic_home" />

</com.example.mycarsystemui.MySystemBarView>
```

---

### 10.9 Step 7: Android.bp

Example:

```bp
android_app {
    name: "MyCarSystemUI",

    srcs: ["src/**/*.java"],

    resource_dirs: ["res"],

    platform_apis: true,

    certificate: "platform",

    privileged: true,

    manifest: "AndroidManifest.xml",

    static_libs: [
        "androidx.annotation_annotation",
    ],
}
```

Important properties:

```bp
platform_apis: true
certificate: "platform"
privileged: true
```

These are important because SystemUI needs access to APIs and permissions normal apps cannot use.

---

### 10.10 Step 8: Add to Product Makefile

You need to include your APK in the product image.

Example:

```makefile
PRODUCT_PACKAGES += MyCarSystemUI
```

This could be in:

```bash
device/<vendor>/<product>/<product>.mk
```

or:

```bash
vendor/<vendor>/<product>/product.mk
```

depending on your project.

---

### 10.11 Step 9: Install as priv-app

Your APK should end up in something like:

```bash
/system_ext/priv-app/MyCarSystemUI/MyCarSystemUI.apk
```

or:

```bash
/system/priv-app/MyCarSystemUI/MyCarSystemUI.apk
```

or:

```bash
/product/priv-app/MyCarSystemUI/MyCarSystemUI.apk
```

For Automotive builds, many OEM apps live under:

```bash
/system_ext/priv-app/
```

Example installation path:

```bash
/system_ext/priv-app/CustomCarSystemUI/CustomCarSystemUI.apk
```

---

## 11. Build and Push SystemUI APK

If you are modifying an existing SystemUI APK, you usually build it like:

```bash
m CustomCarSystemUI
```

or:

```bash
m CarSystemUI
```

or:

```bash
m SystemUI
```

Then locate the output:

```bash
find out/target/product -name "*SystemUI*.apk"
```

Example:

```bash
out/target/product/<target>/system_ext/priv-app/CustomCarSystemUI/CustomCarSystemUI.apk
```

To push it to a device:

```bash
adb root
adb remount
adb shell stop
adb push CustomCarSystemUI.apk /system_ext/priv-app/CustomCarSystemUI/CustomCarSystemUI.apk
adb shell chmod 0644 /system_ext/priv-app/CustomCarSystemUI/CustomCarSystemUI.apk
adb shell chown root:root /system_ext/priv-app/CustomCarSystemUI/CustomCarSystemUI.apk
adb shell restorecon /system_ext/priv-app/CustomCarSystemUI/CustomCarSystemUI.apk
adb reboot
```

Before replacing it, backup the old one:

```bash
adb root
adb remount
adb pull /system_ext/priv-app/CustomCarSystemUI/CustomCarSystemUI.apk ./CustomCarSystemUI_backup.apk
```

Then push the new one.

---

## 12. Common Automotive SystemUI Use Cases

### 12.1 Add New Button to Side Bar

You need to change:

```text
layout XML
drawable icon
view/controller Java or Kotlin code
possibly permissions
```

Example:

```xml
<ImageButton
    android:id="@+id/navigation_button"
    android:layout_width="72dp"
    android:layout_height="72dp"
    android:src="@drawable/ic_navigation" />
```

Then in code:

```java
ImageButton navButton = findViewById(R.id.navigation_button);

navButton.setOnClickListener(v -> {
    Intent intent = getContext().getPackageManager()
            .getLaunchIntentForPackage("com.example.navigation");

    if (intent != null) {
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        getContext().startActivity(intent);
    }
});
```

---

### 12.2 Detect Which App Is on Top

In Automotive SystemUI, you may need to know if Native Map is currently visible.

Conceptually:

```text
Listen to task stack changes
Get top running task
Check topActivity package/class
Update SystemUI state
```

Pseudo-code:

```java
boolean isNativeMap(ComponentName topActivity) {
    return "com.example.map".equals(topActivity.getPackageName())
            && "com.example.map.MainActivity"
                .equals(topActivity.getClassName());
}
```

Then:

```java
if (isNativeMap) {
    showMapBackground();
} else {
    hideMapBackground();
}
```

This is a common pattern for map visibility, navigation apps, and contextual UI behavior in Automotive systems.

---

### 12.3 Change UI Based on Night Mode

You may listen to:

```text
UiModeManager
Configuration changes
CarPropertyManager
VHAL signal
Custom theme signal
```

Example:

```java
@Override
protected void onConfigurationChanged(Configuration newConfig) {
    boolean isNight =
        (newConfig.uiMode & Configuration.UI_MODE_NIGHT_MASK)
        == Configuration.UI_MODE_NIGHT_YES;

    updateTheme(isNight);
}
```

In Automotive, your theme may also come from a car property such as display theme.

Conceptually:

```text
Read VHAL property
Map value to theme
Update SystemUI resources/state
```

---

### 12.4 Show Glow When Map Is Visible

This is common in custom IVI systems.

Flow:

```text
Task changed
↓
Check if Native Map is top activity
↓
Set nativeMapVisible = true
↓
System bars update background
↓
Show edge glow drawables
```

Example structure:

```java
public void setNativeMapVisible(boolean visible) {
    mNativeMapVisible = visible;

    if (visible) {
        mLeftBar.setBackgroundResource(R.drawable.left_map_glow);
        mRightBar.setBackgroundResource(R.drawable.right_map_glow);
    } else {
        mLeftBar.setBackgroundResource(R.drawable.default_bar_background);
        mRightBar.setBackgroundResource(R.drawable.default_bar_background);
    }
}
```

---

## 13. Debugging SystemUI

SystemUI debugging is very important because small mistakes can crash the whole UI.

### 13.1 Check Logs

Use:

```bash
adb logcat | grep -i SystemUI
```

For a custom Automotive SystemUI app:

```bash
adb logcat | grep -i CustomCarSystemUI
```

Or:

```bash
adb logcat | grep -i CarSystemUI
```

Useful filters:

```bash
adb logcat -s SystemUI
adb logcat -s CarSystemUI
adb logcat -s AndroidRuntime
adb logcat -s ActivityTaskManager
adb logcat -s WindowManager
```

---

### 13.2 Check Crash Logs

```bash
adb logcat -b crash
```

Or:

```bash
adb logcat | grep -i "FATAL EXCEPTION"
```

If SystemUI crashes repeatedly, you may see:

```text
FATAL EXCEPTION: main
Process: com.android.systemui
```

or:

```text
Process: com.example.carsystemui
```

---

### 13.3 Restart SystemUI Manually

Depending on package name:

```bash
adb shell pkill com.android.systemui
```

or:

```bash
adb shell pkill com.example.carsystemui
```

Android may restart it automatically.

Sometimes you can use:

```bash
adb shell stop
adb shell start
```

But that restarts the Android framework and is heavier.

---

### 13.4 Check Installed APK Path

```bash
adb shell pm path com.android.systemui
```

or your package:

```bash
adb shell pm path com.example.carsystemui
```

---

### 13.5 Check Permissions

```bash
adb shell dumpsys package com.android.systemui
```

Look for:

```text
grantedPermissions
privileged
signatures
```

---

### 13.6 Check Windows

```bash
adb shell dumpsys window
```

Search for SystemUI windows:

```bash
adb shell dumpsys window | grep -i systemui
```

Or:

```bash
adb shell dumpsys window | grep -i navigation
adb shell dumpsys window | grep -i status
```

---

### 13.7 Check Top Activity

```bash
adb shell dumpsys activity activities | grep -i top
```

Or:

```bash
adb shell dumpsys activity top
```

For Automotive task debugging, also check:

```bash
adb shell dumpsys activity recents
adb shell dumpsys activity activities
```

---

## 14. Best Practices

### 14.1 Do Not Put Heavy Logic in Views

Bad:

```java
CarSystemBarView reads VHAL, checks tasks, changes theme, opens apps
```

Better:

```text
View = display only
Controller = UI behavior
Manager/Repository = system data
```

---

### 14.2 Keep SystemUI Lightweight

SystemUI runs all the time. Heavy work can affect the whole system.

Avoid:

```text
Long loops on main thread
Heavy bitmap processing
Too many animations
Frequent layout invalidation
Repeated task scanning
Memory leaks
```

---

### 14.3 Be Careful with Animations

In Automotive, performance is critical.

If you use Rive, Lottie, GIFs, or animated drawables:

```text
Pause when not visible
Avoid rendering forever
Handle animation end carefully
Do not stop too early if the animation has multiple components
Monitor CPU/GPU usage
```

This is especially important when using complex animations or real-time UI components.

---

### 14.4 Always Test on Real Hardware

Emulator behavior may differ from bench/device behavior.

Especially for:

```text
VHAL signals
HNAV / gRPC
Native Map
Night mode
Display state
Multiple displays
GPU rendering
System bars
```

---

### 14.5 Keep a Backup Before Pushing APK

Always do:

```bash
adb pull /system_ext/priv-app/CustomCarSystemUI/CustomCarSystemUI.apk ./backup.apk
```

before pushing a new one.

---

### 14.6 Add Safe Logs

Use clear logs:

```java
Log.d(TAG, "Native map visible: " + visible);
Log.d(TAG, "Top activity: " + topActivity);
Log.d(TAG, "Updating system bar background");
```

Avoid too many logs in production paths.

---

## 15. Final Mental Model

Think of Automotive SystemUI like this:

```text
Android Framework
    ↓
Starts SystemUI service
    ↓
SystemUI creates system bars and overlays
    ↓
Controllers listen to system/car/app state
    ↓
Views update UI
    ↓
User interacts with bars/buttons
    ↓
SystemUI launches apps or changes system behavior
```

For Automotive specifically:

```text
CarSystemUI = the always-running system UI layer around your IVI apps.
```

It is responsible for:

```text
System bars
Car-specific controls
Multi-display UI
Notification UI
Home/app launcher access
Map/contextual UI integration
Vehicle-aware UI behavior
```

A good learning path for you would be:

```text
1. Understand current CarSystemUI structure
2. Find entry service and Application class
3. Find system bar layouts
4. Find controller classes
5. Trace one button click from XML → View → Controller → action
6. Trace one state change, like Native Map visible → background update
7. Modify a small UI element
8. Add logs
9. Build APK
10. Push to bench
11. Debug crashes/performance
```

The most important files to inspect first are usually:

```bash
AndroidManifest.xml
Android.bp
SystemUIService.java
SystemUIApplication.java
CarSystemUIFactory.java
CarSystemBar.java
CarSystemBarController.java
CarSystemBarView.java
res/layout/*system_bar*.xml
res/values/config.xml
res/values/dimens.xml
res/values/colors.xml
```

## 16. Recommended Practical Exercise

Start with a small debugging exercise inside your current Automotive SystemUI project.

Add a temporary debug `TextView` or logs inside `CarSystemBarView` or `CarSystemBarController` that print:

```text
1. Current top activity
2. Whether Native Map is visible
3. Whether contextual home is active
4. Which display is being updated
```

This exercise teaches you how SystemUI reacts to real system state, which is the heart of Automotive SystemUI development.
