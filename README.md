# react-native-trouble-shooting

> react-native를 개발하면서 마주쳤던 trouble shooting을 모아놓은 레포지토리입니다.

## Android

### 안드로이드 apk 빌드할 때 나는 에러 (Android apk build error)

```bash
$ ./gradlew assembleRelease
...
Execution failed for task ':app:processReleaseResources'.
```

#### Solution

gradle.properties에 아래 코드를 넣는다.

```
android.enableAapt2=false
```

> 주의사항: 하지만 android.enableAapt2 옵션은 2018년 말 이후로 deprecated된다고 하니 나중에는 다른 방법을 찾아야할 것 같다.

참고: https://github.com/facebook/react-native/issues/19239

### 안드로이드 빌드할 때 다음 에러가 뜸

```
Fatal Exception: java.lang.IllegalStateException: Native module RNDeviceModule tried to override RNDeviceModule for module name RNDeviceInfo. If this was your intention, set canOverrideExistingModule=true
       at com.facebook.react.NativeModuleRegistryBuilder.addNativeModule(NativeModuleRegistryBuilder.java:121)
       at com.facebook.react.NativeModuleRegistryBuilder.processPackage(NativeModuleRegistryBuilder.java:109)
       at com.facebook.react.ReactInstanceManager.processPackage(ReactInstanceManager.java:1050)
       at com.facebook.react.ReactInstanceManager.processPackages(ReactInstanceManager.java:1021)
       at com.facebook.react.ReactInstanceManager.createReactContext(ReactInstanceManager.java:959)
       at com.facebook.react.ReactInstanceManager.access$600(ReactInstanceManager.java:109)
       at com.facebook.react.ReactInstanceManager$4.run(ReactInstanceManager.java:802)
       at java.lang.Thread.run(Thread.java:818)
```

#### Solution

MainApplication.java의 protected List<ReactPackage> getPackages() 부분에서 RNDeviceModule이 두 번 들어있을 것이다.

참고: https://github.com/rebeccahughes/react-native-device-info/issues/243

### 안드로이드에서 can’t not find Symbol 에러

mobx 라이브러리 추가한 것이 문제의 발단이었음

#### Solution

JSC가 예전 버전이라 에러가 나는 것 -> JSC를 업데이트해야 한다.

업데이트 하는 방법은 다음과 같다.

1. package.json에 jsc-android 추가 후 npm install 혹은 yarn 명령어로 설치

```
dependencies {
+  "jsc-android": "236355.x.x"
```

2. android/build.gradle에 다음 코드 추가

```gradle
allprojects {
    repositories {
        mavenLocal()
        jcenter()
        maven {
            // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
            url "$rootDir/../node_modules/react-native/android"
        }
+       maven {
+           // Local Maven repo containing AARs with JSC library built for Android
+           url "$rootDir/../node_modules/jsc-android/dist"
+       }
    }
}
```

3. android/app/build.gradle에 다음 코드 추가

```gradle
}
 
+configurations.all {
+    resolutionStrategy {
+        force 'org.webkit:android-jsc:r236355'
+    }
+}
 
dependencies {
    compile fileTree(dir: "libs", include: ["*.jar"])
```

4. rebuild하면 업데이트된 버전의 JSC를 사용할 수 있다.

참고: https://github.com/react-community/jsc-android-buildscripts#how-to-use-it-with-my-react-native-app

### 파이어베이스 관련 안드로이드 빌드 에러

안드로이드에서 빌드하면 다음 에러 발생

```
Program type already present: com.google.android.gms.internal.measurement.zzwp 
```

#### Solution

파이어베이스 코어 라이브러리를 최신 버전으로 올리면 된다.

안드로이드 파이어베이스 sdk의 버전 리스트는 여기를 참고하면 된다.

[링크](https://firebase.google.com/support/release-notes/android#latest_sdk_versions)

참고: https://stackoverflow.com/questions/50146640/android-studio-program-type-already-present-com-google-android-gms-internal-me

## iOS

### iOS 빌드 중 에러 : Duplicate Module Name: react-name

#### Solution

Pods 파일을 제거하고 새로 설치한다.

```bash
cd ios
rm -rf Pods
pod install
```

참고: https://stackoverflow.com/questions/50805753/duplicate-module-name-react-native

### iOS 빌드 중 에러 - Xcode 10: Build input file double-conversion cannot be found

#### Solution

다음 명령어를 프로젝트 루트 경로에서 실행

```bash
$ cd node_modules/react-native/scripts && ./ios-install-third-party.sh && cd ../../../
$ cd node_modules/react-native/third-party/glog-0.3.5/ && ../../scripts/ios-configure-glog.sh && cd ../../../../
```

참고: https://github.com/facebook/react-native/issues/21168

### iOS 빌드 에러: Undefined symbols for architecture armv7

#### Solution

CxxBridge, DoubleConversion, Folly, GLog를 PodFile에 추가

```
# Uncomment the next line to define a global platform for your project
platform :ios, '9.0'
 
target 'reactTest' do
  #Uncomment the next line if you're using Swift or would like to use dynamic frameworks
  #use_frameworks!
  #Pod react-native
  react_native = "../node_modules/react-native"
  pod 'React', :path => react_native, :subspecies => [
  'Core',
  'CxxBridge',
  'DevSupport', # Include this to enable In-App Devmenu if RN >= 0.43
  'RCTText',
  'RCTNetwork',
  'RCTWebSocket', # needed for debugging
  ]
 
  #Pod yoga support for react-native
  pod 'yoga', :path => "#{react_native}/ReactCommon/yoga"
end
```

참고: https://github.com/facebook/react-native/issues/14925

### error: bundling failed: Error: Unable to resolve module `./../react-transform-hmr/lib/index.js`

#### Solution

캐시를 클린해야 한다.

```bash
rm -rf $TMPDIR/react-*; rm -rf $TMPDIR/haste-*; rm -rf $TMPDIR/metro-*; watchman watch-del-all
 
# Start Metro Bundler directly
react-native start
```

참고: https://github.com/facebook/react-native/issues/21490

## Javascript

### decorator 관련 빌드 에러

mobx 라이브러리를 사용하기 위해 decorator를 사용해야 했었는데 빌드 도중 다음 에러가 났다.

```
Error build-assets - new decorators proposal is not supported yet. You must pass the"legacy": true option
```

#### Solution

1. `@babel/plugin-proposal-decorators`를 설치

```bash
npm install --save-dev @babel/plugin-proposal-decorators
```

2. tsconfig.json 파일에 다음 라인을 입력

```json
["@babel/plugin-proposal-decorators", { "legacy": true }],
```

참고: https://github.com/mirumee/saleor/issues/2179


## Contributing

react-native 관련 트러블 슈팅 공유는 언제나 환영입니다!
