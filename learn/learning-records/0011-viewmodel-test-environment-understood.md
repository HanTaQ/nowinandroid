# ViewModel 本地测试环境已理解

学习者已经能够从 `src/test` 判断本地 JVM 测试，理解 `@Before` 隔离每条测试、`MainDispatcherRule` 替换和恢复主线程 dispatcher，并能解释为什么只断言 `stateIn` 的 `initialValue` 时不需要 collector。下一步可以学习通过 Fake Repository 和 collector 驱动 `StateFlow` 发生真实状态变化。
