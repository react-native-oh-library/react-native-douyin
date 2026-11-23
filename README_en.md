> Document Template: v0.4.2

<p align="center">
  <h1 align="center"> <code>react-native-douyin</code> </h1>
</p>

**English** | [简体中文](README.md)

This project is based on [react-native-douyin@1.1.4](https://github.com/wiseqingyang/react-native-douyin/tree/v1.1.4).

The version correspondence details are as follows:

| Name | Version(Npm Address) | Release Information | Supported RN Version | Supported Autolink | Compile API Version | Community Baseline Version | Source code address |
| ---- | -------------------- | ------------------- | -------------------- | ------------------ | ------------------- | -------------------------- | ------------------- |
| @react-native-ohos/react-native-douyin-lib | [~1.2.0](https://www.npmjs.com/package/@react-native-ohos/react-native-douyin-lib) | [Github Releases](https://github.com/react-native-oh-library/react-native-douyin/releases) | 0.72.*/0.77.*/0.82.* | Yes | API12+ | 1.1.4 | [sig](https://github.com/react-native-oh-library/react-native-douyin/tree/sig) |

## Introduction

react-native-douyin is a React Native third-party library based on the Douyin Open Platform OpenSDK. It provides capabilities such as Douyin SDK initialization, authorization login, sharing videos to Douyin, and checking whether Douyin is installed.

## Installation

Go to the project directory and run the following command:

**npm**

```bash
npm install @react-native-ohos/react-native-douyin-lib
```

**yarn**

```bash
yarn add @react-native-ohos/react-native-douyin-lib
```

## Link

| Version | Supported Autolink | Supported RN Version |
|------|--------------------|----------------------|
| ~1.2.0 | Yes | 0.72/0.77/0.82 |

Projects using AutoLink need to be configured according to this document, AutoLink framework guide: https://gitcode.com/CPF-RN/ohos_react_native/blob/main/docs/en/02-development/02-development-guide/autolinking.md

If the version you are using supports Autolink and the project has integrated Autolink, you can skip the ManualLink configuration.
<details>
  <summary>ManualLink: This step provides guidance for manually configuring native dependencies.</summary>

Open the `harmony` directory of the OpenHarmony project in DevEco Studio.

### 1. Overrides RN SDK

To ensure the project relies on the same version of the RN SDK, you need to add an `overrides` field in the project's root `oh-package.json5` file, specifying the RN SDK version to be used. The replacement version can be a specific version number, a semver range, or a locally available HAR package or source directory.

For more information about the purpose of this field, please refer to the [official documentation](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/ide-oh-package-json5-V5#en-us_topic_0000001792256137_overrides).

```json
{
  "overrides": {
    "@rnoh/react-native-openharmony": "file:../node_modules/@react-native-oh/react-native-harmony/react_native_openharmony.har" // a locally available HAR package
    // "@rnoh/react-native-openharmony": "~0.82.18" // ohpm version
    // "@rnoh/react-native-openharmony" : "./react_native_openharmony" // source code directory
  }
}
```

### 2. Introducing Native Code

Currently, two methods are available:

- Use the HAR file.
- Directly link to the source code.

Method 1 (recommended): Use the HAR file.

> [!TIP] The HAR file is stored in the `harmony` directory in the installation path of the third-party library.

Open `entry/oh-package.json5` file and add the following dependencies:

```json
"dependencies": {
    "@react-native-ohos/react-native-douyin-lib": "file:../../node_modules/@react-native-ohos/react-native-douyin-lib/harmony/douyin.har"
  }
```

Click the `sync` button in the upper right corner.

Alternatively, run the following command on the terminal:

```bash
cd entry
ohpm install
```

Method 2: Directly link to the source code.

> [!TIP] For details, see [Directly Linking Source Code](https://gitcode.com/CPF-RN/usage-docs/blob/master/en/link-source-code.md).

### 3. Configuring CMakeLists and Introducing DouYinPackage

Open `entry/src/main/cpp/CMakeLists.txt` and add the following code:

```diff
project(rnapp)
cmake_minimum_required(VERSION 3.4.1)
set(CMAKE_SKIP_BUILD_RPATH TRUE)
set(RNOH_APP_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
set(NODE_MODULES "${CMAKE_CURRENT_SOURCE_DIR}/../../../../../node_modules")
set(OH_MODULE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../../oh_modules")
set(RNOH_CPP_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../../oh_modules/@rnoh/react-native-openharmony/src/main/cpp")
set(RNOH_GENERATED_DIR "${CMAKE_CURRENT_SOURCE_DIR}/generated")
set(LOG_VERBOSITY_LEVEL 1)
set(CMAKE_ASM_FLAGS "-Wno-error=unused-command-line-argument -Qunused-arguments")
set(CMAKE_CXX_FLAGS "-fstack-protector-strong -Wl,-z,relro,-z,now,-z,noexecstack -s -fPIE -pie")
+ set(OH_MODULES "${CMAKE_CURRENT_SOURCE_DIR}/../../../oh_modules")

set(WITH_HITRACE_SYSTRACE 1) # for other CMakeLists.txt files to use
add_compile_definitions(WITH_HITRACE_SYSTRACE)

add_subdirectory("${RNOH_CPP_DIR}" ./rn)

# RNOH_BEGIN: manual_package_linking_1
+ add_subdirectory("${OH_MODULES}/@react-native-ohos/react-native-douyin-lib/harmony/douyin/src/main/cpp" ./douyin)
# RNOH_END: manual_package_linking_1

file(GLOB GENERATED_CPP_FILES "${RNOH_CURRENT_SOURCE_DIR}/generated/*.cpp")

add_library(rnoh_app SHARED
    ${GENERATED_CPP_FILES}
    "./PackageProvider.cpp"
    "${RNOH_CPP_DIR}/RNOHAppNapiBridge.cpp"
)

target_link_libraries(rnoh_app PUBLIC rnoh)

# RNOH_BEGIN: manual_package_linking_2
+ target_link_libraries(rnoh_app PUBLIC rnoh_douyin)
# RNOH_END: manual_package_linking_2
```

Open `entry/src/main/cpp/PackageProvider.cpp` and add the following code:

```diff
#include "RNOH/PackageProvider.h"
+ #include "DouYinPackage.h"

using namespace rnoh;

std::vector<std::shared_ptr<Package>> PackageProvider::getPackages(Package::Context ctx) {
    return {
+     std::make_shared<DouYinPackage>(ctx),
    };
}
```

### 4. Introducing RNDouYinPackage on the ArkTS Side

Open `entry/src/main/ets/RNPackagesFactory.ts` and add the following code:

```diff
  ...
+ import {RNDouYinPackage} from '@react-native-ohos/react-native-douyin-lib/ts';

export function createRNPackages(ctx: RNPackageContext): RNPackage[] {
  return [
+    new RNDouYinPackage(ctx)
  ];
}
```
</details>

### Running

Click the `sync` button in the upper right corner.

Alternatively, run the following command on the terminal:

```bash
cd entry
ohpm install
```

Then build and run the code.

## Constraints

### Compatibility

This document is verified based on the following versions:

1. RNOH: 0.72.96; SDK: HarmonyOS 6.0.0 Release SDK; IDE: DevEco Studio 6.0.0.858; ROM: 6.0.0.112;
2. RNOH: 0.77.18; SDK: HarmonyOS 6.0.0 Release SDK; IDE: DevEco Studio 6.0.0.858; ROM: 6.0.0.112;
3. RNOH: 0.82.7;  SDK: HarmonyOS 6.0.1 Release SDK; IDE: DevEco Studio 6.0.1.260; ROM: 6.0.0.130 SP15;

### Permission Requirements

No special permission is required.

## Example

The following code shows the basic use scenario of the repository:

> \[!WARNING] please refer to the configuration for using the react-native-image-picker library ( https://gitcode.com/CPF-RN/usage-docs/blob/master/en/react-native-image-picker.md ).

> \[!WARNING] The name of the imported repository remains unchanged.

```tsx
import React, { useState } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  SafeAreaView,
  Alert,
  TextInput,
} from 'react-native';
import Douyin from 'react-native-douyin-lib';

import { launchImageLibrary } from 'react-native-image-picker';

const DouyinDemo = () => {
  const [appKey, setAppKey] = useState('');

  const handleInit = () => {
    if (!appKey.trim()) {
      Alert.alert('warning', 'please input AppKey');
      return;
    }
    try {
      Douyin.init(appKey);
      Alert.alert('success', 'douyin SDK initialization completed');
    } catch (error: unknown) {
      Alert.alert('error', 'initialization failed');
    }
  };

  const handleAuth = async () => {
    try {
      const result = await Douyin.auth('user_info', 'random_state_123', 'https://open-client.douyin.com/open_source_demo/test');
      Alert.alert('auth success', JSON.stringify(result));
    } catch (error: unknown) {
      Alert.alert('auth failed', JSON.stringify(error));
    }
  };

  const handleShareVideo = async () => {
    launchImageLibrary({ mediaType: 'video', selectionLimit: 5 }, async (data) => {
      if (data.errorCode) {
        return;
      }
      if (data.assets?.length) {
        try {
          const originalPaths = data.assets
            .map(asset => asset.uri?.replace(/^file:\/\/\//, ''))
            .filter((path): path is string => path !== undefined);
          const result = await Douyin.shareVideo({
            videos: originalPaths,
            isPublish: true,
            title: 'this is title',
            shortTitle: 'this is shortTitle',
          });
          Alert.alert('share success', result);
        } catch (error: unknown) {
          Alert.alert('share failed', JSON.stringify(error));
        }
      }
    });
  };

  return (
    <SafeAreaView style={{ flex: 1 }}>
      <View style={{ padding: 20 }}>
        <TextInput
          style={{ borderWidth: 1, height: 40, paddingHorizontal: 8, marginBottom: 12 }}
          value={appKey}
          onChangeText={setAppKey}
          placeholder="please input AppKey"
        />

        <TouchableOpacity
          style={{ backgroundColor: '#007AFF', padding: 14, borderRadius: 8, marginBottom: 8 }}
          onPress={handleInit}
        >
          <Text style={{ color: '#FFF', textAlign: 'center', fontSize: 16 }}>1. initialization (init)</Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={{ backgroundColor: '#FE2C55', padding: 14, borderRadius: 8, marginBottom: 8 }}
          onPress={handleAuth}
        >
          <Text style={{ color: '#FFF', textAlign: 'center', fontSize: 16 }}>2. Authorized login (auth)</Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={{ backgroundColor: '#25D366', padding: 14, borderRadius: 8 }}
          onPress={handleShareVideo}
        >
          <Text style={{ color: '#FFF', textAlign: 'center', fontSize: 16 }}>3. ShareVideo (shareVideo)</Text>
        </TouchableOpacity>
      </View>
    </SafeAreaView>
  );
};

export default DouyinDemo;
```

## How to Use

**SDK access configuration**

Configure Schemes in `harmony/entry/src/main/module.json5`:

```json
"querySchemes": [
  "snssdk1128",
  "douyinopensdk"
]
```

Configure the associated website domain in `harmony/entry/src/main/module.json5`:

```json
{
  "module": {
    "abilities": [
      {
        "name": "EntryAbility",
        "srcEntry": "./ets/entryability/EntryAbility.ets",
        "icon": "$media:icon",
        "label": "$string:EntryAbility_label",
        // set "exported" to true; if "exported" is false, only system apps with permissions can launch the app, otherwise the app cannot be launched
        "exported": true,
        "startWindowIcon": "$media:icon",
        "startWindowBackground": "$color:start_window_background",
        "skills": [
          {
            "entities": [
              "entity.system.home"
            ],
            "actions": [
              // starting from 5.1.1 (19), it must be configured as "ohos.want.action.home". For versions 5.1.0 (18) and earlier, please configure it as "action.system.home".
              "ohos.want.action.home"
            ]
          },
          {
            "entities": [
              // entities must contain "entity.system.browsable"
              "entity.system.browsable"
            ],
            "actions": [
              // actions must include "ohos.want.action.viewData"
              "ohos.want.action.viewData"
            ],
            "uris": [
              {
                // scheme must be configured as https
                "scheme": "https",
                // host must be configured as an associated URL domain name
                "host": "www.example.com",
                // path is optional, indicating the directory or file path on the domain name server, such as path1 in www.example.com/path1
                // If the application can only handle some specific paths, the paths supported by the application should be configured here to avoid the problem of path links that the application cannot handle being diverted to the application
                "path": "path1"
              }
            ],
            // domainVerify must be set to true
            "domainVerify": true
          }
        ]
      }
    ]
  }
}
```

For the application of website domain names, refer to: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/app-linking-startupapp

**Dependency source configuration**

Add the registry field to the `.ohpmrc` file in the `harmony` directory (create a new one if not available):

```json
registry=https://ohpm.byted.org/repos/ohpm/,http://artifact.bytedance.com/repository/byted-ohpm/
```

**Authorization callback processing**

In the onCreate and onNewWant methods of `harmony/entry/src/main/ets/entryability/EntryAbility.ets`, which set the UIAbility to accept callbacks, call the doResult method to process the callback data.

```ts
import { doResult } from '@react-native-ohos/react-native-douyin-lib';
export default class EntryAbility extends RNAbility {
  onCreate(want: Want) {
    super.onCreate(want)
    // Call the doResult method
    doResult(this.context, want)
  }

  onNewWant(want: Want, _launchParam: AbilityConstant.LaunchParam): void {
    // Call the doResult method
    doResult(this.context, want)
  }
}
```

## Available APIs

> [!TIP] The "Platform" column indicates the platform on which the property is supported in the original third-party library.

> [!TIP] "Yes" means the OpenHarmony platform supports this property; "No" means it is not supported; "Partially" means some capabilities of this property are supported. The usage method is the same across platforms, and the effect is consistent with that of iOS or Android.

### API

| Name | Type | Parameter Type | Return Value | Required | Platform | OpenHarmony Platform Support | Description |
| ---- | ---- | -------------- | ------------ | -------- | -------- | ---------------------------- | ----------- |
| init | function | appKey: string | Promise&lt;void&gt; | no | All | Yes | Initialize the Douyin SDK |
| auth | function | scope: string, state: string, appLink: string | Promise&lt;any&gt; | no | All | Yes | Initiate Douyin authorization login |
| shareVideo | function | config: {videos: string[], isPublish?: boolean, title?: string, shortTitle?: string} | Promise&lt;any&gt; | no | All | Yes | Share videos to Douyin |
| isAppInstalled | function | / | Promise&lt;IApiResponse&gt; | no | All | Yes | Check whether Douyin is installed |

## Known Issues

None

## Other

> [!NOTE] Currently, only video sharing is supported. Sharing images and text is not supported.

> [!NOTE] The isPublish property of the shareVideo interface is currently not supported.

## Directory Structure

````
/react-native-douyin  # Project root directory
├── harmony                       # Adapted third-party library code
│   ├── douyin.har                # HAR package
│   └── douyin                    # Core adapted code
│       ├── index.ets             # Entry file of the adapted code
│       ├── ts.ets                # TypeScript export entry
│       ├── oh-package.json5      # Package management configuration file
│       ├── build-profile.json5   # Build configuration file
│       ├── hvigorfile.ts         # Hvigor build script
│       └── src/main
│           ├── cpp               # C++ adapted code (DouYinPackage, DouYinNativeModule, etc.)
│           ├── ets               # ArkTS adapted code (RNDouYinPackage, RNDouYinTurboModule, generated, etc.)
│           ├── module.json5      # Module configuration file
│           └── resources         # Resource files
├── src                           # RN code
│   ├── RNDouYin.ts               # API wrapper entry
│   └── NativeRNDouYin.ts         # TurboModule Spec declaration
├── index.js                      # Entry file
├── index.d.ts                    # Type declaration file
├── package.json                  # Package configuration file
├── README.OpenSource             # Open source information declaration
├── OAT.xml                       # Open source audit file
├── README.md                     # Chinese installation and usage guide
└── README_en.md                  # English installation and usage guide
````

## How to Contribute

If you find any problems during use, you can submit an [Issue](https://github.com/react-native-oh-library/react-native-douyin/issues). Of course, we also welcome you to submit a [PR](https://github.com/react-native-oh-library/react-native-douyin/pulls).

## License

This project is based on [The MIT License (MIT)](https://github.com/wiseqingyang/react-native-douyin/blob/master/LICENSE). Feel free to enjoy and participate in open source.
