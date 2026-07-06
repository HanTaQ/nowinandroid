# Topic click state loop understood

用户已经能解释 topic 点击事件为什么应写入 repository，而不是直接修改 `onboardingUiState`；也能说明 `Topic` 不包含 followed 状态，因为 followed topics 属于 `UserData`。当前需要进一步巩固的是 Kotlin interface property：`val userData: Flow<UserData>` 是只读属性契约，不是接口内部保存字段。
