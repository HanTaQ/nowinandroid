# Fake 驱动 StateFlow 更新已理解

学习者已经能够解释多个 `stateIn(WhileSubscribed)` 需要各自的 collector，预测缺少 collector 时状态会停留在初始值，并能沿不同依赖路径说明同一次 Fake 输入为何产生 `OnboardingUiState.Loading` 与 `NewsFeedUiState.Success(emptyList())`。同时已经能准确使用 Arrange、Act、Assert 拆解协程状态测试。
