# 维度配置对象抽取为独立共享模块 Spec

## Why

第一个 PR（[#5](https://github.com/LEE20260315/futures-tracker/pull/5)）按任务约束「仅改 `pages/fundamental.html`」将 `FUND_DIMENSION_CONFIG` 放在了页面内联 `<script>` 中。这满足了"配置对象独立、放文件顶部"的要求，但配置仍与覆写逻辑耦合在同一个 HTML 文件里，存在三个不足：

1. **不可复用**：`signal.html` 的基本面因子（`getFundamentalComposite`）如果未来想引用同一套指标→维度映射，无法 import 内联脚本中的变量。
2. **不可测试**：配置对象与 DOM 覆写逻辑混在一起，无法单独对配置做结构校验/单元测试。
3. **维护成本**：调整权重/品种映射需要编辑 887 行的 HTML 文件，容易误碰 DOM 结构。

本 spec 把 `FUND_DIMENSION_CONFIG` 及其前缀匹配辅助抽到独立 `shared/fund-dimension-config.js`，`fundamental.html` 仅保留覆写逻辑并通过 `<script src>` 引入配置。

## What Changes

- **新建 `shared/fund-dimension-config.js`**：导出 `FUND_DIMENSION_CONFIG`（dimensions / indicatorToDimension / varietyTemplates / templates / refuseThreshold）+ 预排序的 `mapIndicatorToDim(name)` 前缀匹配函数 + `validateConfig()` 结构校验函数（开发态 console.warn）。
- **修改 `pages/fundamental.html`**：
  - 在 `shared/ui-core.js` 之后、内联覆写脚本之前新增 `<script src="../shared/fund-dimension-config.js"></script>`。
  - 从内联脚本中**删除** `FUND_DIMENSION_CONFIG` 定义、`_PREFIX_ENTRIES` 预排序、`mapIndicatorToDim` 函数（改为引用全局 `window.FUND_DIMENSION_CONFIG` / `window.mapIndicatorToDim`）。
  - 保留覆写逻辑（`loadFundamental` / `updateFundTotal` / `renderExternalFundSignal`）与解析辅助函数（`parseVarietySection` 等）不变。
- **不改动**：`shared/app-core.js`、`shared/ui-core.js`、`shared/fundamental-feed.json`、后台采集逻辑、信号引擎行为。

## Impact

- **Affected code**：
  - `shared/fund-dimension-config.js`（新建）
  - `pages/fundamental.html`（删除内联配置定义 ~50 行，新增 1 行 `<script src>`）
- **不改动**：`shared/app-core.js`、`shared/ui-core.js`、`shared/fundamental-feed.json`、`signal.html`、后台采集。
- **行为不变**：纯重构，页面渲染/计分/告警去重/新鲜度角标行为与 PR #5 完全一致。
- **Affected specs**：`refactor-fundamental-scoring-by-source`（其"配置对象"需求由本 spec 进一步细化——从"内联独立 config"升级为"独立共享模块"）。

## ADDED Requirements

### Requirement: 维度配置共享模块
系统 SHALL 在 `shared/fund-dimension-config.js` 中定义并全局暴露 `FUND_DIMENSION_CONFIG`（含 dimensions / indicatorToDimension / varietyTemplates / templates / refuseThreshold）与 `mapIndicatorToDim(name)` 前缀匹配函数。配置结构须与 PR #5 中的定义完全一致，不得改变字段名/权重/映射关系。

#### Scenario: 配置从共享模块加载
- **WHEN** `fundamental.html` 加载 `shared/fund-dimension-config.js` 后
- **THEN** `window.FUND_DIMENSION_CONFIG` 与 `window.mapIndicatorToDim` 可用
- **AND** 内联覆写脚本引用这两个全局变量，行为与 PR #5 一致

#### Scenario: 前缀匹配按长度降序
- **WHEN** 调用 `mapIndicatorToDim('DOMESTIC_INV_SR')`
- **THEN** 返回 `'inventory'`（而非 `'supply'`），因 `DOMESTIC_INV` 前缀长于 `ONI` 等

### Requirement: 配置结构校验
`shared/fund-dimension-config.js` SHALL 导出 `validateConfig()` 函数，在开发态（`location.hostname === 'localhost'` 或 `127.0.0.1`）对配置做结构校验：维度 key 唯一、模板权重非负、品种映射指向已定义模板、indicatorToDimension 前缀无冲突。校验失败时 `console.warn` 输出问题项，不阻断渲染。

#### Scenario: 开发态校验通过
- **WHEN** 在 localhost 打开页面
- **THEN** `validateConfig()` 执行且无 `console.warn` 输出
- **AND** 生产环境（非 localhost）不执行校验

## MODIFIED Requirements

### Requirement: 维度模板配置对象（升级为共享模块）
原 `refactor-fundamental-scoring-by-source` spec 要求"在 `pages/fundamental.html` 顶部内联脚本中定义独立 config 对象 `FUND_DIMENSION_CONFIG`"。本 spec 将其升级为：`FUND_DIMENSION_CONFIG` 定义在 `shared/fund-dimension-config.js` 中，`fundamental.html` 通过 `<script src>` 引入。内联脚本仅保留覆写逻辑与 DOM 解析辅助。

### Requirement: fundamental.html 内联脚本范围
`pages/fundamental.html` 的内联 `<script>` SHALL 仅包含 `FTRender` 方法覆写（`loadFundamental` / `updateFundTotal` / `renderExternalFundSignal`）与 `fullReport` 解析辅助函数（`parseVarietySection` / `parseActiveIndicators` / `parseMissingItems` / `parseWarehouseReceipt` / `computeDimScore` / `formatIndicatorNote` / `groupIndicatorsByDim` / `setDim`）。`FUND_DIMENSION_CONFIG` 定义、`_PREFIX_ENTRIES` 预排序、`mapIndicatorToDim` 函数 SHALL 移至 `shared/fund-dimension-config.js`。

## REMOVED Requirements

无。本 spec 是纯重构，不移除任何功能。
