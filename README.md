# aauto-sdk

This is an unofficial SDK for Android Auto.

## Note

With great power comes great responsibility.
Do not write apps that distract drivers.
Distraction is dangerous.

## Demo

Have a look at [the demo app](https://github.com/martoreto/aauto-sdk-demo).

## Usage

Add this to your main _build.gradle_:
```gradle
allprojects {
    repositories {
        maven { url "https://jitpack.io" }
    }
}
```

and this to you app's _build.gradle_::

```gradle
dependencies {
    compile 'com.github.martoreto:aauto-sdk:v4.0'
}
```
