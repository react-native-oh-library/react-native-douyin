> 文档模板：v0.4.2

<p align="center">
  <h1 align="center"> <code>react-native-douyin</code> </h1>
</p>

本项目基于 [react-native-douyin@1.1.4](https://github.com/wiseqingyang/react-native-douyin/tree/v1.1.4) 开发。

版本所属关系如下：

| 三方库名称 | 三方库版本（npm地址） | 发布信息 | 支持RN版本 | Autolink | 编译API版本 | 社区基线版本 | 源码地址 |
| ------------ | ---------------------- | -------- | ---------- | -------- | ------------ | ------------ | -------- |
| @react-native-ohos/react-native-douyin-lib | [~1.2.0](https://www.npmjs.com/package/@react-native-ohos/react-native-douyin-lib) | [Github Releases](https://github.com/react-native-oh-library/react-native-douyin/releases) | 0.72.*/0.77.*/0.82.* | 是 | API12+ | 1.1.4 | [sig](https://github.com/react-native-oh-library/react-native-douyin/tree/sig) |

## 简介

react-native-douyin 是基于抖音开放平台 OpenSDK 的 React Native 三方库，提供抖音 SDK 初始化、授权登录、分享视频到抖音、检测抖音是否安装等能力。

## 下载安装

进入到工程目录并输入以下命令：

**npm**

```bash
npm install @react-native-ohos/react-native-douyin-lib
```

**yarn**

```bash
yarn add @react-native-ohos/react-native-douyin-lib
```

## Link

| 版本 | 是否支持autolink | RN框架版本 |
|------|----------------|-----------|
| ~1.2.0 | 是 | 0.72/0.77/0.82 |

使用AutoLink的工程需要根据该文档配置，Autolink框架指导文档：https://gitcode.com/CPF-RN/ohos_react_native/blob/main/docs/zh-cn/02-开发/02-开发指南/Autolinking.md

如您使用的版本支持 Autolink，并且工程已接入 Autolink，可跳过ManualLink配置。
<details>
  <summary>ManualLink: 此步骤为手动配置原生依赖项的指导</summary>

首先需要使用 DevEco Studio 打开项目里的 HarmonyOS 工程 `harmony`。

### 1. Overrides RN SDK

为了让工程依赖同一个版本的 RN SDK，需要在工程根目录的 `oh-package.json5` 添加 overrides 字段，指向工程需要使用的 RN SDK 版本。替换的版本既可以是一个具体的版本号，也可以是一个模糊版本，还可以是本地存在的 HAR 包或源码目录。

关于该字段的作用请阅读[官方说明](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-oh-package-json5-V5#zh-cn_topic_0000001792256137_overrides)

```json
{
  "overrides": {
    "@rnoh/react-native-openharmony": "file:../node_modules/@react-native-oh/react-native-harmony/react_native_openharmony.har" // 指向本地 har 包的路径
    // "@rnoh/react-native-openharmony": "~0.82.18" // ohpm 在线版本
    // "@rnoh/react-native-openharmony" : "./react_native_openharmony" // 指向源码路径
  }
}
```

### 2. 引入原生端代码

目前有两种方法：

- 通过 har 包引入；
- 直接链接源码。

方法一：通过 har 包引入（推荐）

> [!TIP] har 包位于三方库安装路径的 `harmony` 文件夹下。

打开 `entry/oh-package.json5`，添加以下依赖

```json
"dependencies": {
    "@react-native-ohos/react-native-douyin-lib": "file:../../node_modules/@react-native-ohos/react-native-douyin-lib/harmony/douyin.har"
  }
```

点击右上角的 `sync` 按钮

或者在命令行终端执行：

```bash
cd entry
ohpm install
```

方法二：直接链接源码

> [!TIP] 如需使用直接链接源码，请参考[直接链接源码说明](https://gitcode.com/CPF-RN/usage-docs/blob/master/zh-cn/link-source-code.md)

### 3. 配置 CMakeLists 和引入 DouYinPackage

打开 `entry/src/main/cpp/CMakeLists.txt`，添加：

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

打开 `entry/src/main/cpp/PackageProvider.cpp`，添加：

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

### 4. 在 ArkTs 侧引入 RNDouYinPackage

打开 `entry/src/main/ets/RNPackagesFactory.ts`，添加：

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

### 运行

点击右上角的 `sync` 按钮

或者在命令行终端执行：

```bash
cd entry
ohpm install
```

然后编译、运行即可。

## 约束与限制

### 兼容性

本文档内容基于以下版本验证通过：

1. RNOH: 0.72.96; SDK: HarmonyOS 6.0.0 Release SDK; IDE: DevEco Studio 6.0.0.858; ROM: 6.0.0.112;
2. RNOH: 0.77.18; SDK: HarmonyOS 6.0.0 Release SDK; IDE: DevEco Studio 6.0.0.858; ROM: 6.0.0.112;
3. RNOH: 0.82.7;  SDK: HarmonyOS 6.0.1 Release SDK; IDE: DevEco Studio 6.0.1.260; ROM: 6.0.0.130 SP15;

### 权限要求

无特殊权限要求。

## 使用示例

下面的代码展示了这个库的基本使用场景：

> \[!WARNING] react-native-image-picker 库的使用配置请参考 (https://gitcode.com/CPF-RN/usage-docs/blob/master/zh-cn/react-native-image-picker.md) 。

> \[!WARNING] 使用时 import 的库名不变。

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
      Alert.alert('提示', '请输入 AppKey');
      return;
    }
    try {
      Douyin.init(appKey);
      Alert.alert('成功', '抖音 SDK 初始化完成');
    } catch (error: unknown) {
      Alert.alert('错误', '初始化失败');
    }
  };

  const handleAuth = async () => {
    try {
      const result = await Douyin.auth('user_info', 'random_state_123', 'https://open-client.douyin.com/open_source_demo/test');
      Alert.alert('授权成功', JSON.stringify(result));
    } catch (error: unknown) {
      Alert.alert('授权失败', JSON.stringify(error));
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
          Alert.alert('分享成功', result);
        } catch (error: unknown) {
          Alert.alert('分享失败', JSON.stringify(error));
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
          placeholder="请输入 AppKey"
        />

        <TouchableOpacity
          style={{ backgroundColor: '#007AFF', padding: 14, borderRadius: 8, marginBottom: 8 }}
          onPress={handleInit}
        >
          <Text style={{ color: '#FFF', textAlign: 'center', fontSize: 16 }}>1. 初始化 (init)</Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={{ backgroundColor: '#FE2C55', padding: 14, borderRadius: 8, marginBottom: 8 }}
          onPress={handleAuth}
        >
          <Text style={{ color: '#FFF', textAlign: 'center', fontSize: 16 }}>2. 授权登录 (auth)</Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={{ backgroundColor: '#25D366', padding: 14, borderRadius: 8 }}
          onPress={handleShareVideo}
        >
          <Text style={{ color: '#FFF', textAlign: 'center', fontSize: 16 }}>3. 分享视频 (shareVideo)</Text>
        </TouchableOpacity>
      </View>
    </SafeAreaView>
  );
};

export default DouyinDemo;
```

## 使用说明

**SDK 接入配置**

在 `harmony/entry/src/main/module.json5` 中配置 Schemes：

```json
"querySchemes": [
  "snssdk1128",
  "douyinopensdk"
]
```

在 `harmony/entry/src/main/module.json5` 中配置关联的网址域名：

```json
{
  "module": {
    "abilities": [
      {
        "name": "EntryAbility",
        "srcEntry": "./ets/entryability/EntryAbility.ets",
        "icon": "$media:icon",
        "label": "$string:EntryAbility_label",
        // 请将exported配置为true；如果exported为false，仅具有权限的系统应用能够拉起该应用，否则无法拉起应用
        "exported": true,
        "startWindowIcon": "$media:icon",
        "startWindowBackground": "$color:start_window_background",
        "skills": [
          {
            "entities": [
              "entity.system.home"
            ],
            "actions": [
              // 从5.1.1(19)开始，须配置为"ohos.want.action.home"。对于5.1.0(18)及之前版本，请配置为"action.system.home"。
              "ohos.want.action.home"
            ]
          },
          {
            "entities": [
              // entities必须包含"entity.system.browsable"
              "entity.system.browsable"
            ],
            "actions": [
              // actions必须包含"ohos.want.action.viewData"
              "ohos.want.action.viewData"
            ],
            "uris": [
              {
                // scheme须配置为https
                "scheme": "https",
                // host须配置为关联的网址域名
                "host": "www.example.com",
                // path可选，表示域名服务器上的目录或文件路径，例如www.example.com/path1中的path1
                // 如果应用只能处理部分特定的path，则此处应该配置应用所支持的path，避免出现应用不能处理的path链接也被引流到应用中的问题
                "path": "path1"
              }
            ],
            // domainVerify须设置为true
            "domainVerify": true
          }
        ]
      }
    ]
  }
}
```

网址域名申请请参考: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/app-linking-startupapp

**依赖源配置**

在 `harmony` 目录下的 `.ohpmrc` 文件（如果没有请新建）中的 registry 字段增加 [http://artifact.bytedance.com/repository/byted-ohpm/]：

```json
registry=https://ohpm.byted.org/repos/ohpm/,http://artifact.bytedance.com/repository/byted-ohpm/
```

**授权回调处理**

在 `harmony/entry/src/main/ets/entryability/EntryAbility.ets` 设置接受回调的 UIAbility 的 onCreate 和 onNewWant 方法中，调用 doResult 方法处理回调数据。

```ts
import { doResult } from '@react-native-ohos/react-native-douyin-lib';
export default class EntryAbility extends RNAbility {
  onCreate(want: Want) {
    super.onCreate(want)
    //调用doResult方法
    doResult(this.context, want)
  }

  onNewWant(want: Want, _launchParam: AbilityConstant.LaunchParam): void {
   //调用doResult方法
    doResult(this.context, want)
  }
}
```

## 接口说明

> [!TIP] "Platform"列表示该属性在原三方库上支持的平台。

> [!TIP] "OpenHarmony平台支持"列为 "是" 表示 OpenHarmony 平台支持该属性；"否" 则表示不支持；"部分支持" 表示仅实现了功能的一部分。使用方法跨平台一致，效果对标 iOS 或 Android 的效果。

### API

| 名称 | 类型 | 参数类型 | 返回值 | 必填 | 平台 | OpenHarmony平台支持 | 描述 |
| ---- | ---- | -------- | ------ | ---- | ---- | ------------------- | ---- |
| init | function | appKey: string | Promise&lt;void&gt; | no | All | 是 | 初始化抖音 SDK |
| auth | function | scope: string, state: string, appLink: string | Promise&lt;any&gt; | no | All | 是 | 发起抖音授权登录 |
| shareVideo | function | config: {videos: string[], isPublish?: boolean, title?: string, shortTitle?: string} | Promise&lt;any&gt; | no | All | 是 | 分享视频到抖音 |
| isAppInstalled | function | / | Promise&lt;IApiResponse&gt; | no | All | 是 | 检测抖音是否安装 |

## 遗留问题

无

## 其他

> [!注意事项] 当前只支持分享视频，不支持分享图文。

> [!注意事项] shareVideo接口的isPublish当前不支持。

## 目录结构

````
/react-native-douyin  # 项目根目录
├── harmony                       # 已适配的三方库代码
│   ├── douyin.har                # har包
│   └── douyin                    # 适配核心代码
│       ├── index.ets             # 适配代码入口
│       ├── ts.ets                # TypeScript 导出入口
│       ├── oh-package.json5      # 包管理配置文件
│       ├── build-profile.json5   # 构建配置文件
│       ├── hvigorfile.ts         # Hvigor 构建脚本
│       └── src/main
│           ├── cpp               # C++ 适配代码（DouYinPackage、DouYinNativeModule 等）
│           ├── ets               # ArkTS 适配代码（RNDouYinPackage、RNDouYinTurboModule、generated 等）
│           ├── module.json5      # 模块配置文件
│           └── resources         # 资源文件
├── src                           # RN代码
│   ├── RNDouYin.ts               # API 封装入口
│   └── NativeRNDouYin.ts         # TurboModule Spec 声明
├── index.js                      # 入口文件
├── index.d.ts                    # 类型声明文件
├── package.json                  # 包配置文件
├── README.OpenSource             # 开源信息声明
├── OAT.xml                       # 开源审计文件
├── README.md                     # 中文安装使用方法
└── README_en.md                  # 英文安装使用方法
````

## 贡献代码

使用过程中发现任何问题都可以提交 [Issue](https://github.com/react-native-oh-library/react-native-douyin/issues)，当然，也非常欢迎提交 [PR](https://github.com/react-native-oh-library/react-native-douyin/pulls) 。

## 开源协议

本项目基于 [The MIT License (MIT)](https://github.com/wiseqingyang/react-native-douyin/blob/master/LICENSE) ，请自由地享受和参与开源。
