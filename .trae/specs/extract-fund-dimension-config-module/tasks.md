# Tasks

> 范围：新建 `shared/fund-dimension-config.js`，从 `pages/fundamental.html` 内联脚本中抽出 `FUND_DIMENSION_CONFIG` 及前缀匹配辅助。纯重构，行为不变。

- [x] Task 1: 新建 `shared/fund-dimension-config.js`
  - [x] 1.1 将 `FUND_DIMENSION_CONFIG` 对象（dimensions / indicatorToDimension / varietyTemplates / templates / refuseThreshold）从 `fundamental.html` 内联脚本原样迁移到 `shared/fund-dimension-config.js`
  - [x] 1.2 迁移 `_PREFIX_ENTRIES` 预排序逻辑与 `mapIndicatorToDim(name)` 函数
  - [x] 1.3 将 `FUND_DIMENSION_CONFIG`、`mapIndicatorToDim` 挂到 `window` 全局
  - [x] 1.4 新增 `validateConfig()` 函数：校验维度 key 唯一、模板权重非负、品种映射指向已定义模板、前缀无冲突；仅在 localhost/127.0.0.1 执行，失败时 `console.warn`
  - [x] 1.5 文件末尾自动调用 `validateConfig()`（开发态）

- [x] Task 2: 修改 `pages/fundamental.html` 引用共享模块
  - [x] 2.1 在 `<script src="../shared/ui-core.js"></script>` 之后新增 `<script src="../shared/fund-dimension-config.js"></script>`
  - [x] 2.2 从内联脚本中删除 `FUND_DIMENSION_CONFIG` 定义（约 lines 435-472）
  - [x] 2.3 从内联脚本中删除 `_PREFIX_ENTRIES` 预排序与 `mapIndicatorToDim` 函数（约 lines 474-532）
  - [x] 2.4 确认内联脚本中所有引用 `FUND_DIMENSION_CONFIG` / `mapIndicatorToDim` 的位置改为读取全局变量

- [x] Task 3: 行为回归验证（Node.js 逻辑验证 75/75 PASS）
  - [x] 3.1 验证 `shared/fund-dimension-config.js` 可独立加载（vm.runInContext 模拟浏览器）
  - [x] 3.2 验证配置结构与 PR #5 完全一致（逐字段比对 dimensions/indicatorToDimension/varietyTemplates/templates/refuseThreshold）
  - [x] 3.3 验证 `mapIndicatorToDim` 前缀匹配（17 个用例：ONI→supply、FRED_2YR_YIELD→macro、DOMESTIC_INV_SR→inventory 等）
  - [x] 3.4 验证 `validateConfig()` 返回空数组（无问题）
  - [x] 3.5 验证 `fundamental.html` 内联脚本已删除配置定义、`_PREFIX_ENTRIES`、`mapIndicatorToDim` 函数
  - [x] 3.6 验证 `fundamental.html` 保留全部覆写逻辑与解析辅助函数
  - [x] 3.7 确认 `window.FUND_DIMENSION_CONFIG` 与 `window.mapIndicatorToDim` 在全局可用
  - [x] 3.8 确认 localhost 下 `validateConfig()` 无 `console.warn` 输出

# Task Dependencies
- Task 2 依赖 Task 1（共享模块需先存在）
- Task 3 依赖 Task 1 + Task 2
