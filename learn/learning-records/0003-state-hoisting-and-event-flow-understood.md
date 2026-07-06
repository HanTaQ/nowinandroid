# State hoisting and event flow understood

用户已经能解释 `SingleTopicButton` 中多个控件共享同一个 `isSelected` source of truth，能区分 `Surface.onClick` 与 `NiaIconToggleButton.onCheckedChange` 的触发范围，并理解业务状态不应保存在组件内部的 `remember mutableStateOf` 中，而应通过 ViewModel/repository 写入数据源后再由 UI state 回流。
