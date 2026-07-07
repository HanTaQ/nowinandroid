# StateIn exercise exposed Flow operator gap

用户已经理解 `initialValue = Loading` 的 UI 意义，也知道测试中需要订阅 `StateFlow` 才能触发 `WhileSubscribed` 上游收集。但当前对 `Flow.map`、`List.map`、`combine`、`flatMapLatest` 等操作符的类型变化还缺少体系，需要用 Now in Android 源码按类型链继续练习。
