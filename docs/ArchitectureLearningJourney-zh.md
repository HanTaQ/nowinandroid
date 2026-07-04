# 架构学习旅程

在这个学习旅程中，你将学习 Now in Android 应用的架构：它的分层、关键类，以及这些类之间如何交互。


## 目标和要求

应用架构的目标是：



*   尽可能遵循 [官方架构指南](https://developer.android.com/jetpack/guide)。
*   让开发者容易理解，不采用过于实验性的方案。
*   支持多名开发者在同一个代码库中协作。
*   方便在开发者本机和持续集成（CI）中进行本地测试与 instrumented tests。
*   尽量缩短构建时间。


## 架构概览

应用架构分为三层：[数据层](https://developer.android.com/jetpack/guide/data-layer)、[领域层](https://developer.android.com/jetpack/guide/domain-layer) 和 [UI 层](https://developer.android.com/jetpack/guide/ui-layer)。


<center>
<img src="images/architecture-1-overall.png" width="600px" alt="展示整体应用架构的图" />
</center>

> [!NOTE]
> Android 官方架构不同于其他架构，例如 “Clean Architecture”。其他架构中的概念不一定适用于这里，或者会以不同方式应用。[这里有更多讨论](https://github.com/android/nowinandroid/discussions/1273)。

这个架构遵循响应式编程模型，并使用[单向数据流](https://developer.android.com/jetpack/guide/ui-layer#udf)。在数据层位于底部的前提下，关键概念是：



*   上层响应下层的变化。
*   事件向下流动。
*   数据向上流动。

数据流通过 stream 实现，在本项目中具体使用 [Kotlin Flows](https://developer.android.com/kotlin/flow)。


### 示例：在 For You 页面显示新闻

当应用第一次运行时，它会尝试从远程服务器加载新闻资源列表。当选择 `prod` build flavor 时会使用远程服务器；`demo` build 会使用本地数据。加载完成后，这些内容会根据用户选择的兴趣展示给用户。

下面的图展示了为实现这一点会发生哪些事件，以及数据如何在相关对象之间流动。


![展示新闻资源如何显示在 For You 页面上的图](images/architecture-2-example.png "展示新闻资源如何显示在 For You 页面上的图")


下面是每一步发生的事情。查找对应代码最简单的方法是把项目加载到 Android Studio，然后搜索 Code 列中的文本。实用快捷方式：按两次 <kbd>SHIFT</kbd>。

<table>
  <tr>
   <td><strong>步骤</strong>
   </td>
   <td><strong>说明</strong>
   </td>
   <td><strong>代码</strong>
   </td>
  </tr>
  <tr>
   <td>1
   </td>
   <td>应用启动时，会将一个用于同步所有 repositories 的 <a href="https://developer.android.com/topic/libraries/architecture/workmanager">WorkManager</a> 任务加入队列。
   </td>
   <td><code>Sync.initialize</code>
   </td>
  </tr>
  <tr>
   <td>2
   </td>
   <td><code>ForYouViewModel</code> 调用 <code>GetUserNewsResourcesUseCase</code>，获取一个包含新闻资源及其 bookmarked/saved 状态的 stream。直到 user repository 和 news repository 都发出一个 item 之前，这个 stream 不会发出任何 item。在等待期间，feed state 会被设置为 <code>Loading</code>。
   </td>
   <td>搜索 <code>NewsFeedUiState.Loading</code> 的使用位置
   </td>
  </tr>
  <tr>
   <td>3
   </td>
   <td>user data repository 会从一个由 Proto DataStore 支持的本地数据源中获取 <code>UserData</code> 对象 stream。
   </td>
   <td><code>NiaPreferencesDataSource.userData</code>
   </td>
  </tr>
  <tr>
   <td>4
   </td>
   <td>WorkManager 执行同步任务。这个任务会调用 <code>OfflineFirstNewsRepository</code>，开始与远程数据源同步数据。
   </td>
   <td><code>SyncWorker.doWork</code>
   </td>
  </tr>
  <tr>
   <td>5
   </td>
   <td><code>OfflineFirstNewsRepository</code> 调用 <code>RetrofitNiaNetwork</code>，使用 <a href="https://square.github.io/retrofit/">Retrofit</a> 执行真实的 API 请求。
   </td>
   <td><code>OfflineFirstNewsRepository.syncWith</code>
   </td>
  </tr>
  <tr>
   <td>6
   </td>
   <td><code>RetrofitNiaNetwork</code> 调用远程服务器上的 REST API。
   </td>
   <td><code>RetrofitNiaNetwork.getNewsResources</code>
   </td>
  </tr>
  <tr>
   <td>7
   </td>
   <td><code>RetrofitNiaNetwork</code> 接收来自远程服务器的网络响应。
   </td>
   <td><code>RetrofitNiaNetwork.getNewsResources</code>
   </td>
  </tr>
  <tr>
   <td>8
   </td>
   <td><code>OfflineFirstNewsRepository</code> 通过在本地 <a href="https://developer.android.com/training/data-storage/room">Room database</a> 中插入、更新或删除数据，将远程数据同步到 <code>NewsResourceDao</code>。
   </td>
   <td><code>OfflineFirstNewsRepository.syncWith</code>
   </td>
  </tr>
  <tr>
   <td>9
   </td>
   <td>当 <code>NewsResourceDao</code> 中的数据发生变化时，它会被发送到新闻资源数据 stream 中。这个 stream 是一个 <a href="https://developer.android.com/kotlin/flow">Flow</a>。
   </td>
   <td><code>NewsResourceDao.getNewsResources</code>
   </td>
  </tr>
  <tr>
   <td>10
   </td>
   <td><code>OfflineFirstNewsRepository</code> 在这个 stream 上充当一个 <a href="https://developer.android.com/kotlin/flow#modify">intermediate operator</a>，将传入的 <code>PopulatedNewsResource</code> 转换为公开的 <code>NewsResource</code> model。<code>PopulatedNewsResource</code> 是数据库 model，属于数据层内部；<code>NewsResource</code> 会被其他层消费。
   </td>
   <td><code>OfflineFirstNewsRepository.getNewsResources</code>
   </td>
  </tr>
  <tr>
   <td>11
   </td>
   <td><code>GetUserNewsResourcesUseCase</code> 将新闻资源列表与用户数据结合，发出一个 <code>UserNewsResource</code> 列表。
   </td>
   <td><code>GetUserNewsResourcesUseCase.invoke</code>
   </td>
  </tr>
  <tr>
   <td>12
   </td>
   <td>当 <code>ForYouViewModel</code> 收到这些可保存的新闻资源时，它会将 feed state 更新为 <code>Success</code>。

   </td>
   <td>搜索 <code>NewsFeedUiState.Success</code> 的实例
   </td>
  </tr>
</table>



## 数据层

数据层以 offline-first 的方式实现，作为应用数据和业务逻辑的来源。它是应用中所有数据的 source of truth。



![展示数据层架构的图](images/architecture-3-data-layer.png "展示数据层架构的图")


每个 repository 都有自己的 models。例如，`TopicsRepository` 有一个 `Topic` model，`NewsRepository` 有一个 `NewsResource` model。

Repositories 是其他层访问数据的公开 API，它们提供了访问应用数据的 _唯一_ 方式。Repositories 通常会提供一个或多个读取和写入数据的方法。


### 读取数据

数据以 data streams 的形式暴露。这意味着 repository 的每个客户端都必须准备好响应数据变化。数据不会以快照形式暴露，例如 `getModel`，因为无法保证快照在被使用时仍然有效。

读取操作以本地存储作为 source of truth，因此从 `Repository` 实例读取数据时通常不预期发生错误。不过，在尝试协调本地存储和远程来源的数据时，可能会发生错误。关于错误协调的更多内容，请查看下方的数据同步部分。

_示例：读取 topic 列表_

可以通过订阅 `TopicsRepository::getTopics` flow 来获取 Topics 列表。这个 flow 会发出 `List<Topic>`。

每当 topics 列表发生变化时，例如新增了一个 topic，更新后的 `List<Topic>` 就会被发送到 stream 中。


### 写入数据

为了写入数据，repository 会提供 suspend functions。调用者需要确保这些函数的执行被放在合适的 scope 中。

_示例：关注一个 topic_

只需要调用 `UserDataRepository.toggleFollowedTopicId`，传入用户想关注的 topic ID，并传入 `followed=true` 表示应该关注该 topic。传入 `false` 则表示取消关注。


### 数据源

一个 repository 可能依赖一个或多个数据源。例如，`OfflineFirstTopicsRepository` 依赖以下数据源：


<table>
  <tr>
   <td><strong>名称</strong>
   </td>
   <td><strong>底层支持</strong>
   </td>
   <td><strong>用途</strong>
   </td>
  </tr>
  <tr>
   <td>TopicsDao
   </td>
   <td><a href="https://developer.android.com/training/data-storage/room">Room/SQLite</a>
   </td>
   <td>与 Topics 相关的持久化关系型数据
   </td>
  </tr>
  <tr>
   <td>NiaPreferencesDataSource
   </td>
   <td><a href="https://developer.android.com/topic/libraries/architecture/datastore">Proto DataStore</a>
   </td>
   <td>与用户偏好相关的持久化非结构化数据，具体来说是用户感兴趣的 Topics。这些数据在 .proto 文件中使用 protobuf 语法定义和建模。
   </td>
  </tr>
  <tr>
   <td>NiaNetworkDataSource
   </td>
   <td>通过 Retrofit 访问的远程 API
   </td>
   <td>topics 数据，通过 REST API endpoints 以 JSON 形式提供。
   </td>
  </tr>
</table>



### 数据同步

Repositories 负责协调本地存储中的数据与远程来源中的数据。一旦从远程数据源获取数据，它就会立刻被写入本地存储。更新后的数据会从本地存储（Room）发送到相关的数据 stream 中，并被所有正在监听的客户端接收。

这种方式确保应用的读取和写入职责彼此分离，不会相互干扰。

如果数据同步过程中发生错误，会采用指数退避策略。这个策略通过 `SyncWorker` 委托给 `WorkManager` 执行，`SyncWorker` 是 `Synchronizer` 接口的一个实现。

可以查看 `OfflineFirstNewsRepository.syncWith`，它是数据同步的一个示例。

## 领域层

[领域层](https://developer.android.com/topic/architecture/domain-layer)包含 use cases。这些类包含一个单独的可调用方法，即 `operator fun invoke`，其中承载业务逻辑。

这些 use cases 用于简化 ViewModels，并移除其中重复的逻辑。它们通常会组合并转换来自 repositories 的数据。

例如，`GetUserNewsResourcesUseCase` 会将来自 `NewsRepository` 的 `NewsResource` stream 与来自 `UserDataRepository` 的 `UserData` stream 组合起来，创建一个 `UserNewsResource` stream。这些 stream 使用 `Flow` 实现。这个 stream 会被多个 ViewModels 使用，用于在屏幕上展示新闻资源及其 bookmarked 状态。

值得注意的是，目前 Now in Android 的领域层 _没有_ 包含用于事件处理的 use cases。事件由 UI 层直接调用 repositories 上的方法来处理。

## UI Layer

[UI 层](https://developer.android.com/topic/architecture/ui-layer)包含：



*   使用 [Jetpack Compose](https://developer.android.com/jetpack/compose) 构建的 UI 元素
*   [Android ViewModels](https://developer.android.com/topic/libraries/architecture/viewmodel)

ViewModels 从 use cases 和 repositories 接收数据 streams，并将它们转换为 UI state。UI 元素反映这些 state，并为用户提供与应用交互的方式。这些交互会作为 events 传递给 ViewModel，并在那里被处理。


![展示 UI 层架构的图](images/architecture-4-ui-layer.png "展示 UI 层架构的图")


### 建模 UI state

UI state 使用接口和不可变 data classes 建模为 sealed hierarchy。State objects 只会通过数据 streams 的转换被发出。这种方式确保：



*   UI state 始终代表底层应用数据，应用数据是 source-of-truth。
*   UI 元素会处理所有可能的 state。

**示例：For You 页面上的新闻 feed**

For You 页面上的新闻资源 feed，也就是列表，使用 `NewsFeedUiState` 建模。这是一个 sealed interface，会创建两个可能 state 组成的层级：



*   `Loading` 表示数据正在加载
*   `Success` 表示数据已经成功加载。Success state 包含新闻资源列表。

`feedState` 会传递给 `ForYouScreen` composable，由它处理这两种 state。


### 将 streams 转换为 UI state

ViewModels 会从一个或多个 use cases 或 repositories 接收作为 cold [flows](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/index.html) 的数据 streams。它们会被 [combined](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/combine.html) 组合在一起，或者简单地 [mapped](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/map.html)，最终生成一个 UI state flow。这个单一 flow 随后会使用 [stateIn](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/state-in.html) 转换为 hot flow。转换为 state flow 以后，UI 元素就可以读取 flow 中最后已知的 state。

**示例：显示已关注的 topics**

`InterestsViewModel` 将 `uiState` 暴露为 `StateFlow<InterestsUiState>`。这个 hot flow 通过获取 `GetFollowableTopicsUseCase` 提供的 `List<FollowableTopic>` cold flow 创建。每当发出一个新列表时，它都会被转换为 `InterestsUiState.Interests` state，并暴露给 UI。


### 处理用户交互

用户操作通过普通方法调用从 UI 元素传递给 ViewModels。这些方法会作为 lambda expressions 传递给 UI 元素。

**示例：关注一个 topic**

`InterestsScreen` 接收一个名为 `followTopic` 的 lambda expression，它由 `InterestsViewModel.followTopic` 提供。每当用户点击一个 topic 进行关注时，就会调用这个方法。然后 ViewModel 会通过通知 user data repository 来处理这个操作。


## 延伸阅读

[应用架构指南](https://developer.android.com/topic/architecture)

[Jetpack Compose](https://developer.android.com/jetpack/compose)
