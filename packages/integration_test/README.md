# integration_test

This package enables self-driving testing of Flutter code on devices and emulators.
It adapts flutter_test results into a format that is compatible with `flutter drive`
and native Android instrumentation testing.

## Usage

Add a dependency on the `integration_test` and `flutter_test` package in the
`dev_dependencies` section of pubspec.yaml. For plugins, do this in the
pubspec.yaml of the example app.

Invoke `IntegrationTestWidgetsFlutterBinding.ensureInitialized()` at the start
of a test file, e.g.

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();
  testWidgets("failing test example", (WidgetTester tester) async {
    expect(2 + 2, equals(5));
  });
}
```

## Test locations

It is recommended to put integration_test tests in the `test/` folder of the app
or package. For example apps, if the integration_test test references example
app code, it should go in `example/test/`. It is also acceptable to put
integration_test tests in `test_driver/` folder so that they're alongside the
runner app (see below).

## Using Flutter Driver to Run Tests

`IntegrationTestWidgetsTestBinding` supports launching the on-device tests with
`flutter drive`. Note that the tests don't use the `FlutterDriver` API, they
use `testWidgets` instead.

Put the a file named `<package_name>_integration_test.dart` in the app'
`test_driver` directory:

```dart
import 'dart:async';

import 'package:integration_test/integration_test_driver.dart';

Future<void> main() async => integrationDriver();
```

To run a example app test with Flutter driver:

```sh
cd example
flutter drive test/<package_name>_integration.dart
```

To test plugin APIs using Flutter driver:

```sh
cd example
flutter drive --driver=test_driver/<package_name>_test.dart test/<package_name>_integration_test.dart
```

You can run tests on web in release or profile mode.

First you need to make sure you have downloaded the driver for the browser.

```sh
cd example
flutter drive -v --target=test_driver/<package_name>dart -d web-server --release --browser-name=chrome
```

## Android device testing

Create an instrumentation test file in your application's
**android/app/src/androidTest/java/com/example/myapp/** directory (replacing
com, example, and myapp with values from your app's package name). You can name
this test file MainActivityTest.java or another name of your choice.

```java
package com.example.myapp;

import androidx.test.rule.ActivityTestRule;
import dev.flutter.plugins.integration_test.FlutterTestRunner;
import org.junit.Rule;
import org.junit.runner.RunWith;

@RunWith(FlutterTestRunner.class)
public class MainActivityTest {
  @Rule
  public ActivityTestRule<MainActivity> rule = new ActivityTestRule<>(MainActivity.class, true, false);
}
```

Update your application's **myapp/android/app/build.gradle** to make sure it
uses androidx's version of AndroidJUnitRunner and has androidx libraries as a
dependency.

```gradle
android {
  ...
  defaultConfig {
    ...
    testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
  }
}

dependencies {
    testImplementation 'junit:junit:4.12'

    // https://developer.android.com/jetpack/androidx/releases/test/#1.2.0
    androidTestImplementation 'androidx.test:runner:1.2.0'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'
}
```

To run a test on a local Android device (emulated or physical):

```sh
./gradlew app:connectedAndroidTest -Ptarget=`pwd`/../test_driver/<package_name>_integration_test.dart
```

## Firebase Test Lab

If this is your first time testing with Firebase Test Lab, you'll need to follow
the guides in the [Firebase test lab
documentation](https://firebase.google.com/docs/test-lab/?gclid=EAIaIQobChMIs5qVwqW25QIV8iCtBh3DrwyUEAAYASAAEgLFU_D_BwE)
to set up a project.

To run a test on Android devices using Firebase Test Lab, use gradle commands to build an
instrumentation test for Android, after creating `androidTest` as suggested in the last section.

```bash
pushd android
# flutter build generates files in android/ for building the app
flutter build apk
./gradlew app:assembleAndroidTest
./gradlew app:assembleDebug -Ptarget=<path_to_test>.dart
popd
```

Upload the build apks Firebase Test Lab, making sure to replace <PATH_TO_KEY_FILE>,
<PROJECT_NAME>, <RESULTS_BUCKET>, and <RESULTS_DIRECTORY> with your values.

```bash
gcloud auth activate-service-account --key-file=<PATH_TO_KEY_FILE>
gcloud --quiet config set project <PROJECT_NAME>
gcloud firebase test android run --type instrumentation \
  --app build/app/outputs/apk/debug/app-debug.apk \
  --test build/app/outputs/apk/androidTest/debug/app-debug-androidTest.apk\
  --timeout 2m \
  --results-bucket=<RESULTS_BUCKET> \
  --results-dir=<RESULTS_DIRECTORY>
```

You can pass additional parameters on the command line, such as the
devices you want to test on. See
[gcloud firebase test android run](https://cloud.google.com/sdk/gcloud/reference/firebase/test/android/run).

## iOS device testing

You need to change `iOS/Podfile` to avoid test target statically linking to the plugins. One way is to
link all of the plugins dynamically:

```
target 'Runner' do
  use_frameworks!
  ...
end
```

To run a test on your iOS device (simulator or real), rebuild your iOS targets with Flutter tool.

```sh
flutter build ios -t test_driver/<package_name>_integration_test.dart (--simulator)
```

Open Xcode project (by default, it's `ios/Runner.xcodeproj`). Create a test target
(navigating `File > New > Target...` and set up the values) and a test file `RunnerTests.m` and
change the code. You can change `RunnerTests.m` to the name of your choice.

```objective-c
#import <XCTest/XCTest.h>
#import <integration_test/IntegrationTestIosTest.h>

INTEGRATION_TEST_IOS_RUNNER(RunnerTests)
```

Now you can start RunnerTests to kick out integration tests!