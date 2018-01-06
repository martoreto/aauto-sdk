# aauto-sdk

This is an unofficial SDK for Android Auto.

## Note

With great power comes great responsibility.
Do not write apps that distract drivers.
Distraction is dangerous.

## Demo

Have a look at these example apps:

* [aauto-sdk-demo](https://github.com/martoreto/aauto-sdk-demo)

* [aastats](https://github.com/martoreto/aastats)

## Usage

Add this to your main _build.gradle_:
```gradle
allprojects {
    repositories {
        maven { url "https://jitpack.io" }
    }
}
```

and this to your app's _build.gradle_:

```gradle
dependencies {
    compile 'com.github.martoreto:aauto-sdk:v4.0'
}
```

and this to your _AndroidManifest.xml_ inside `<application>`:

```xml
<meta-data
    android:name="com.google.android.gms.car.application"
    android:resource="@xml/automotive_app_desc" />
```

and save this as _res/xml/automotive_app_desc.xml_:

```xml
<?xml version="1.0" encoding="utf-8"?>
<automotiveApp xmlns:tools="http://schemas.android.com/tools">
    <uses name="service" tools:ignore="InvalidUsesTagAttribute" />
    <uses name="projection" tools:ignore="InvalidUsesTagAttribute" />
    <uses name="notification" />
</automotiveApp>
```

## API

There are no docs, but you can explore the API in Android Studio after switching to the _Package_ view:

![screenshot1](media/screenshot1.png)

Then have a look in the following places under _Libraries_:

```
com.google.android.apps.auto.sdk
android.support.car
values/color.xml
values/dimens.xml
values/styles.xml
```

Actually, some version of `android.support.car` [is open source](https://android.googlesource.com/platform/packages/services/Car/+/master/car-support-lib/) and has docs in code.

### OEM activities (right-most icon)

look like this in _AndroidMainfest.xml_:

```xml
<service
    android:name=".CarService"
    android:label="@string/car_service_name"
    tools:ignore="ExportedService">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="com.google.android.gms.car.category.CATEGORY_PROJECTION" />
        <category android:name="com.google.android.gms.car.category.CATEGORY_PROJECTION_OEM" />
    </intent-filter>
</service>
```

### Accessing sensors

Here's some example code for reading sensor data provided by car.

This can be used in an Android Auto activity, or a background service.

Some documentation: [CarSensorManager.java](https://android.googlesource.com/platform/packages/services/Car/+/master/car-support-lib/src/android/support/car/hardware/CarSensorManager.java)

```java
import android.support.car.Car;
import android.support.car.CarConnectionCallback;
import android.support.car.hardware.CarSensorEvent;
import android.support.car.hardware.CarSensorManager;

@Override
public void onStart() {
    mCar = Car.createCar(this, mCarConnectionCallback);
    mCar.connect();
}

@Override
public void onStop() {
    try {
        mCar.disconnect();
    } catch (Exception e) {
        Log.w(TAG, "Error disconnecting from car", e);
    }
}

private final CarConnectionCallback mCarConnectionCallback = new CarConnectionCallback() {
    @Override
    public void onConnected(Car car) {
        try {
            Log.d(TAG, "Connected to car");
            CarSensorManager sensorManager = (CarSensorManager) car.getCarManager(Car.SENSOR_SERVICE);
            sensorManager.addListener(mSensorsListener, CarSensorManager.SENSOR_TYPE_DRIVING_STATUS,
                    CarSensorManager.SENSOR_RATE_NORMAL);
            CarSensorEvent ds = sensorManager.getLatestSensorEvent(CarSensorManager.SENSOR_TYPE_DRIVING_STATUS);
            if (ds != null) {
                mSensorsListener.onSensorChanged(sensorManager, ds);
            } else {
                Log.i(TAG, "Driving status unknown.");
            }
        } catch (Exception e) {
            Log.w(TAG, "Error setting up car connection", e);
        }
    }

    @Override
    public void onDisconnected(Car car) {
        Log.d(TAG, "Disconnected from car");
    }
};

private final CarSensorManager.OnSensorChangedListener mSensorsListener = new CarSensorManager.OnSensorChangedListener() {
    @Override
    public void onSensorChanged(CarSensorManager sensorManager, CarSensorEvent ev) {
        Log.v(TAG, "Sensor event: " + ev);
        CarSensorEvent.DrivingStatusData ds = ev.getDrivingStatusData();
        if (ds != null) {
            // Do something with this
        }
    }
};
```

### Vendor channel

Android Auto protocol defines another communication method between the car and an app running on the phone,
called vendor channel. It's basically a byte channel.

The protocol inside is vendor-specific:

 * Volkswagen Group uses [Exlap](https://www.scribd.com/document/158754515/EXLAP-Specification-V1-3-Creative-Commons-BY-SA-3-0-Volkswagen-pdf)
 * Ford Performance vehicles use [an unknown protocol](https://m.apkpure.com/ford-performance-app/com.ford.performance.android.experience)

Example use:

```java
CarVendorExtensionManagerLoader vexLoader =
        (CarVendorExtensionManagerLoader)mCar.getCarManager(
                CarVendorExtensionManagerLoader.VENDOR_EXTENSION_LOADER_SERVICE);
mVexManager = vexLoader.getManager(getVendorChannelName());
if (mVexManager == null) {
    throw new RuntimeException("Vendor channel not available");
}
mVexManager.registerListener(new CarVendorExtensionManager.CarVendorExtensionListener() {
    @Override
    public void onData(CarVendorExtensionManager carVendorExtensionManager, byte[] bytes) {
        // Do something with data
    }
});
mVexManager.sendData(new byte[] { (byte) 1, (byte) 2, (byte) 3 });
```

In order to use the vendor channel:

 * the package name of the application must be on the list provided by the car,

 * the app must have

   ```
   <uses-permission android:name="com.google.android.gms.permission.CAR_VENDOR_EXTENSION"/>
   ```
   in its _AndroidManifest.xml_ (and granted at runtime on Marshmallow or later).
   
## Building

Usually, there's no need to build the SDK yourself. If you have a need, please share it by
contributing pull requests. :)

To build the SDK yourself:

1. Set ``ANDROID_HOME`` enviroment variable to point to the Android SDK folder.

2. Do this:

    ```sh
    ./gradlew build
    ```

The result is ``build/aauto.aar``.
