# DataStore update transaction understood

用户已经能解释 `DataStore.updateData` 作为事务性 read-modify-write 操作的含义，理解 follow topic 时会更新 DataStore 中的 followed topic map，失败时捕获 `IOException`，也理解只改 ViewModel 内存状态会丢失持久化并破坏应用重启后的用户状态。
