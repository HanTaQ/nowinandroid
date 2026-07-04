# Mission: Learn Modern Android With Now in Android

## Why
通过 Android 官方示例 Now in Android，系统掌握现代 Android 应用的架构、模块化、测试、UI 与性能实践。目标不是只读懂文档，而是能把这些模式迁移到自己的 Android 项目中，知道为什么这样分层、这样拆模块、这样测试和优化。

## Success looks like
- 能从一个功能入口追踪 UI -> ViewModel -> use case -> repository -> local/remote data source 的完整路径。
- 能解释 `app`、`feature:*:api`、`feature:*:impl`、`core:*`、`sync:*`、`benchmarks` 的职责边界。
- 能为 ViewModel、repository、Compose UI、截图测试和仪器测试选择合适的测试方式。
- 能读懂 Now in Android 的 Compose UI 结构、设计系统、主题、适配布局和 Navigation 组织方式。
- 能把读源码时遇到的 Compose layout、state、modifier、Flow、StateFlow 等基础概念拆开理解，而不是只靠猜。
- 能说明 Baseline Profile、Macrobenchmark、Compose compiler metrics 在性能优化中的位置。

## Constraints
- 以当前仓库 `D:\T5Migration\NowInAndroid` 为主要教材，优先读真实代码和官方文档。
- 每次学习聚焦一个小主题，并配一个可完成的小练习，避免一次性铺太多概念。
- 主要使用中文讲解，关键 Android/Kotlin/Gradle/Compose/Flow 术语保留英文名称。
- 遇到 Compose 或 Kotlin Flow 源码卡点时，先补必要的体系知识，再回到 Now in Android 的架构主线。

## Out of scope
- 暂不把 Now in Android 改造成另一个业务 App。
- 暂不深入后端服务实现，因为 `prod` 后端不是公开学习重点。
- 暂不以发布上架为目标，只把构建、测试和性能流程作为学习对象。
