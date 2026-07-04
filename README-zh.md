![Now in Android](docs/images/nia-splash.jpg "Now in Android")

<a href="https://play.google.com/store/apps/details?id=com.google.samples.apps.nowinandroid"><img src="https://play.google.com/intl/en_us/badges/static/images/badges/en_badge_web_generic.png" height="70"></a>

Now in Android 应用
==================

**了解这个应用是如何设计和构建的：[设计案例研究](https://goo.gle/nia-figma)、[架构学习旅程](docs/ArchitectureLearningJourney-zh.md) 和 [模块化学习旅程](docs/ModularizationLearningJourney.md)。**

这是 [Now in Android](https://developer.android.com/series/now-in-android) 应用的代码仓库。它仍然是一个**持续开发中的项目**。

**Now in Android** 是一个完全使用 Kotlin 和 Jetpack Compose 构建的完整 Android 应用。它遵循 Android 设计与开发最佳实践，目标是为开发者提供一个有用的参考项目。作为一个可运行的应用，它通过定期提供新闻更新，帮助开发者跟上 Android 开发领域的最新动态。

这个应用目前仍在开发中。`prodRelease` variant 已经[在 Play Store 上架](https://play.google.com/store/apps/details?id=com.google.samples.apps.nowinandroid)。

# 功能

**Now in Android** 展示来自 [Now in Android](https://developer.android.com/series/now-in-android) 系列的内容。用户可以浏览近期视频、文章和其他内容的链接。用户也可以关注自己感兴趣的主题，并在有匹配其关注主题的新内容发布时收到通知。

## 截图

![显示 For You 页面、Interests 页面和 Topic 详情页面的截图](docs/images/screenshots.png "显示 For You 页面、Interests 页面和 Topic 详情页面的截图")

# 开发环境

**Now in Android** 使用 Gradle 构建系统，可以直接导入 Android Studio。请确保你使用的是[最新稳定版 Android Studio](https://developer.android.com/studio)。

将运行配置切换为 `app`。

![image](https://user-images.githubusercontent.com/873212/210559920-ef4a40c5-c8e0-478b-bb00-4879a8cf184a.png)

可以构建并运行 `demoDebug` 和 `demoRelease` build variants。`prod` variants 会使用一个后端服务器，而这个后端目前没有公开提供。

![image](https://user-images.githubusercontent.com/873212/210560507-44045dc5-b6d5-41ca-9746-f0f7acf22f8e.png)

当项目可以正常运行后，你可以参考下面的学习旅程，更深入地了解项目使用了哪些库和工具，为什么 UI、测试、架构等部分采用这些方案，以及这些不同部分如何组合成一个完整的应用。

# 架构

**Now in Android** 应用遵循 [官方架构指南](https://developer.android.com/topic/architecture)，并在 [架构学习旅程](docs/ArchitectureLearningJourney-zh.md) 中进行了详细说明。

# 模块化

**Now in Android** 应用已经完全模块化。你可以在 [模块化学习旅程](docs/ModularizationLearningJourney.md) 中找到关于本项目模块化策略的详细指导和说明。

# 构建

这个应用包含常见的 `debug` 和 `release` build variants。

此外，`app` 的 `benchmark` variant 用于测试启动性能并生成 baseline profile。更多信息见下文。

`app-nia-catalog` 是一个独立应用，用于展示为 **Now in Android** 定制样式的组件列表。

这个应用还使用 [product flavors](https://developer.android.com/studio/build/build-variants#product-flavors) 来控制应用内容的加载来源。

`demo` flavor 使用静态本地数据，因此可以立即构建并探索 UI。

`prod` flavor 会真实请求后端服务器，以提供最新内容。目前还没有公开可用的后端。

普通开发使用 `demoDebug` variant。UI 性能测试使用 `demoRelease` variant。

# 测试

为了让各个组件更容易测试，**Now in Android** 使用 [Hilt](https://developer.android.com/training/dependency-injection/hilt-android) 做依赖注入。

大多数数据层组件都定义为接口。然后，带有各种依赖的具体实现会被绑定到这些接口上，供应用中的其他组件使用。

在测试中，**Now in Android** 一个值得注意的做法是：它不使用任何 mocking library。相反，生产实现可以通过 Hilt 的测试 API 替换为 test doubles；对于 `ViewModel` 测试，也可以通过手动构造函数注入来替换依赖。

这些 test doubles 实现了与生产实现相同的接口，并且通常提供一个更简单但仍然接近真实情况的实现，同时带有额外的测试 hook。

这样得到的测试不那么脆弱，也可能覆盖更多生产代码，而不是只验证对 mock 的某些调用。

示例：

- 在 instrumentation tests 中，会使用一个临时文件夹来保存用户偏好设置，并在每个测试结束后清空。
  这样可以使用真实的 `DataStore`，并覆盖所有相关代码，而不是 mock 数据更新的 flow。

- 每个 repository 都有对应的 `Test` 实现。这些实现会实现普通的、完整的 repository 接口，并提供测试专用 hook。
  `ViewModel` 测试会使用这些 `Test` repositories，因此可以通过测试 hook 操作 `Test` repository 的状态，并验证由此产生的行为，而不是检查某个 repository 方法是否被调用。

运行测试可以执行以下 Gradle 任务：

- `testDemoDebug`：针对 `demoDebug` variant 运行所有本地测试。截图测试会失败，原因见下文。为了避免这个问题，可以在运行单元测试前先执行 `recordRoborazziDemoDebug`。
- `connectedDemoDebugAndroidTest`：针对 `demoDebug` variant 运行所有 instrumented tests。

> [!NOTE]
> 不应该运行 `./gradlew test` 或 `./gradlew connectedAndroidTest`，因为这会对 _所有_ build variants 执行测试。这样既没有必要，也会导致失败，因为目前只有 `demoDebug` variant 支持测试。其他 variants 目前没有任何测试，虽然未来可能会改变。

## 截图测试

截图测试会截取应用中某个屏幕或 UI 组件的截图，并将其与之前录制的、已知渲染正确的截图进行比较。

例如，Now in Android 有一些[截图测试](https://github.com/android/nowinandroid/blob/main/app/src/testDemo/kotlin/com/google/samples/apps/nowinandroid/ui/NiaAppScreenSizesScreenshotTests.kt)，用于验证不同屏幕尺寸下导航是否正确显示。对应的[已知正确截图](https://github.com/android/nowinandroid/tree/main/app/src/testDemo/screenshots)也存放在仓库中。

Now in Android 使用 [Roborazzi](https://github.com/takahirom/roborazzi) 运行部分屏幕和 UI 组件的截图测试。处理截图测试时，以下 Gradle 任务很有用：

- `verifyRoborazziDemoDebug`：运行所有截图测试，并与已知正确截图进行校验。
- `recordRoborazziDemoDebug`：录制新的“已知正确”截图。当你修改了 UI，并且已经手动确认它们渲染正确时，可以使用这个命令。截图会存储在 `modulename/src/test/screenshots`。
- `compareRoborazziDemoDebug`：在测试失败时生成失败截图与已知正确截图之间的对比图片。这些图片同样可以在 `modulename/src/test/screenshots` 中找到。

> [!NOTE]
> **关于截图测试失败的说明**
> 本仓库中存储的已知正确截图是在 CI 中使用 Linux 录制的。其他平台可能会生成略有不同的图片，因此截图测试可能失败。
> 在非 Linux 平台开发时，一个变通方法是在开始工作前，在 `main` 分支上运行 `recordRoborazziDemoDebug`。完成修改后，`verifyRoborazziDemoDebug` 就只会识别真正由改动引起的差异。

关于截图测试的更多信息，可以[查看这个演讲](https://www.droidcon.com/2023/11/15/easy-screenshot-testing-with-compose/)。

# UI

这个应用使用 [Material 3 guidelines](https://m3.material.io/) 进行设计。你可以在 [Now in Android Material 3 Case Study](https://goo.gle/nia-figma) 中了解更多设计过程并获取设计文件。设计资源也[提供 PDF 版本](docs/Now-In-Android-Design-File.pdf)。

屏幕和 UI 元素全部使用 [Jetpack Compose](https://developer.android.com/jetpack/compose) 构建。

这个应用有两套主题：

- Dynamic color：如果设备支持，会根据[用户当前的系统配色主题](https://material.io/blog/announcing-material-you)生成颜色。
- Default theme：当 dynamic color 不可用时，使用预定义颜色。

每套主题也都支持 dark mode。

这个应用使用 adaptive layouts 来[支持不同屏幕尺寸](https://developer.android.com/guide/topics/large-screens/support-different-screen-sizes)。

可以在[这里了解更多 UI 架构](docs/ArchitectureLearningJourney-zh.md#ui-layer)。

# 性能

## Benchmarks

所有使用 [`Macrobenchmark`](https://developer.android.com/topic/performance/benchmarking/macrobenchmark-overview) 编写的测试都位于 `benchmarks` 模块中。这个模块还包含用于生成 Baseline profile 的测试。

## Baseline profiles

这个应用的 baseline profile 位于 [`app/src/main/baseline-prof.txt`](app/src/main/baseline-prof.txt)。
它包含一些规则，使应用启动时关键用户路径可以进行 AOT 编译。
想了解更多 baseline profiles，可以阅读[这份文档](https://developer.android.com/studio/profile/baselineprofiles)。

> [!NOTE]
> 如果 release build 触及会改变应用启动过程的代码，就需要重新生成 baseline profile。

要生成 baseline profile，请选择 `benchmark` build variant，并在 AOSP Android Emulator 上运行 `BaselineProfileGenerator` benchmark test。
然后将模拟器中生成的 baseline profile 复制到 [`app/src/main/baseline-prof.txt`](app/src/main/baseline-prof.txt)。

## Compose compiler metrics

运行下面的命令可以获取并分析 Compose compiler metrics：

```bash
./gradlew assembleRelease -PenableComposeCompilerMetrics=true -PenableComposeCompilerReports=true
```

reports 文件会被添加到 [build/compose-reports](build/compose-reports)。metrics 文件也会被添加到 [build/compose-metrics](build/compose-metrics)。

关于 Compose compiler metrics 的更多信息，可以查看[这篇博客文章](https://medium.com/androiddevelopers/jetpack-compose-stability-explained-79c10db270c8)。

# License

**Now in Android** 基于 Apache License Version 2.0 分发。更多信息见 [license](LICENSE)。
