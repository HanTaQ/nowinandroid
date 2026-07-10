# Now in Android Modern Android Resources

## Knowledge

- [Now in Android README](../README.md)
  项目总览。用来理解 demo/prod flavor、测试命令、UI、性能和官方学习入口。
- [Architecture Learning Journey](../docs/ArchitectureLearningJourney.md)
  架构主教材。用来学习 data/domain/UI layers、UDF、Flow、offline-first repository、ViewModel UI state。
- [Modularization Learning Journey](../docs/ModularizationLearningJourney.md)
  模块化主教材。用来学习 `app`、`feature:*:api`、`feature:*:impl`、`core:*` 的职责和依赖规则。
- [Android official architecture guidance](https://developer.android.com/topic/architecture)
  官方架构原则。用来校准 Now in Android 为什么采用分层、UDF、repository、ViewModel。
- [Android modularization guidance](https://developer.android.com/topic/modularization)
  官方模块化原则。用来理解模块边界、依赖方向、构建速度和团队协作取舍。
- [Jetpack Compose documentation](https://developer.android.com/jetpack/compose)
  UI 主资源。用来学习 Compose state、layout、Material 3、testing 和性能注意事项。
- [Compose state documentation](https://developer.android.com/develop/ui/compose/state)
  用来补足 Compose 状态管理、recomposition、state hoisting、Flow 到 Compose State 的基础词汇。
- [Compose modifiers documentation](https://developer.android.com/develop/ui/compose/modifiers)
  用来理解 `Modifier` 链、尺寸、padding、modifier 顺序，以及源码里的 layout 写法。
- [Compose constraints and modifier order](https://developer.android.com/develop/ui/compose/layouts/constraints-modifiers)
  用来理解 `heightIn`、`sizeIn`、父子约束、测量和布局阶段。
- [Compose lazy lists and lazy grids](https://developer.android.com/develop/ui/compose/lists)
  用来理解 `LazyVerticalStaggeredGrid`、`LazyHorizontalGrid`、`items`、`key` 等列表/网格 API。
- [Kotlin Flow combine API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/combine.html)
  用来理解多个 Flow 如何组合为一个 UI state Flow。
- [Kotlin Flow guide](https://kotlinlang.org/docs/flow.html)
  用来建立 Flow、Flow operators、cold flow、collection 的基础模型。
- [Kotlin Flow stateIn API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/state-in.html)
  用来理解 ViewModel 里为什么把 cold Flow 转成 `StateFlow`，以及 `initialValue` 的意义。
- [Android testing documentation](https://developer.android.com/training/testing)
  测试主资源。用来对照本项目的 unit tests、instrumented tests、Hilt test doubles、Roborazzi screenshot tests。
- [Macrobenchmark overview](https://developer.android.com/topic/performance/benchmarking/macrobenchmark-overview)
  性能测试主资源。用来理解 `benchmarks` 模块和启动性能测试。
- [Baseline Profiles documentation](https://developer.android.com/studio/profile/baselineprofiles)
  性能优化主资源。用来理解 `app/src/main/baseline-prof.txt` 和 release 构建中的 profile 生成。

- [Testing Kotlin flows on Android](https://developer.android.com/kotlin/flow/test)
  Android 官方 Flow 测试指南。用于学习 fake producer、断言 `StateFlow.value`，以及测试由 `stateIn` 创建的 `StateFlow` 时为什么需要 collector。
- [kotlinx-coroutines-test API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-test/)
  Kotlin 官方协程测试 API。用于查询 `runTest`、`TestScope`、`TestDispatcher`、虚拟时间和 `Dispatchers.setMain`。

## Wisdom (Communities)

- [Android Developers official site](https://developer.android.com/)
  官方知识入口。遇到架构、测试、Compose、性能概念分歧时优先回到这里核对。
- [android/nowinandroid GitHub Discussions](https://github.com/android/nowinandroid/discussions)
  项目相关讨论。用来查看官方示例背后的取舍，例如架构和模块化争议。
- [Android Developers Blog](https://android-developers.googleblog.com/)
  官方实践更新。用来跟进 Compose、性能、架构组件的新推荐。

## Gaps

- 后续需要根据你的进度补充更细的主题资源，例如 Flow/Turbine、Hilt testing、Roborazzi、Compose stability、Navigation 3。
