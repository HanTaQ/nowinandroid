# TopicSelection source reading progress

用户已经能解释 `rememberLazyGridState()` 保留 lazy grid 滚动状态、`GridCells.Fixed(3)` 需要稳定高度约束，以及 topic 点击事件应交给 ViewModel 处理。当前需要进一步校正的是 Compose lazy item `key`：它更主要用于标识 item 身份、保留对应 composition/state/位置关系，而不是直接等同于 RecyclerView 的 View 回收复用。
