# 残留优化点修复 Spec

## Why

P0/P1/P2 主修复已全部落地并通过核查，但遗留 5 个优化点：清除 Token 按钮缺失、三路降级失败提示不明确、Gist 恢复无冲突确认、日志搜索无清空快捷键、仪表盘盈亏曲线仍为占位。本 spec 一次性收尾这 5 项，使系统达到完整可用状态。

## What Changes

### P1: 清除 Token 按钮
- **settings.html** Gist 同步配置区新增"清除 Token"按钮
- 点击调用 `removeSecure('githubToken')` + `removeSecure('gistId')`，清空 Token 输入框，toast 提示"Token 已从本地清除"

### P2-1: 三路降级失败提示细化
- **app-core.js** `fetchPricesNow` 全部失败分支：toast 显示具体失败原因（futsseapi 超时/东财无响应/新浪超时），状态指示灯改红色
- 追踪每路失败原因到 `fetchFailReasons` 数组，拼接展示

### P2-2: Gist 恢复冲突确认
- **app-core.js** `restoreFromGist`：恢复前对比本地 `state.lastBackup` 与 Gist 数据的 `lastBackup` 时间戳
- 若本地更新则 `confirm()` 弹窗确认"本地数据更新，确认用 Gist 版本覆盖吗？"
- Gist 数据无 lastBackup 时直接恢复不确认

### P3-1: 日志搜索清空按钮
- **journal.html** 搜索过滤栏右侧新增"× 清空"按钮
- 点击重置三个过滤器（品种/类型/关键词）并重新渲染

### P3-2: 仪表盘盈亏曲线接真实数据
- **app-core.js** `updateEquityHistory`：平仓时自动追加资金曲线节点（当前仅每日更新一次）
- **ui-core.js** `renderEquityChart`：增强图表——X 轴日期标签、Y 轴金额标签、起始资金基线、当前净值标注
- **ui-core.js** `renderDashboard`：若 `equityHistory` 不足 2 条但 `closedTrades` 有数据，用 closedTrades 构建临时曲线

## Impact

- Affected code:
  - `pages/settings.html`（清除 Token 按钮）
  - `pages/journal.html`（清空筛选按钮）
  - `shared/app-core.js`（fetchPricesNow 失败提示细化；restoreFromGist 冲突确认；updateEquityHistory 平仓追加节点）
  - `shared/ui-core.js`（renderEquityChart 增强；renderDashboard 临时曲线兜底）

## ADDED Requirements

### Requirement: 清除 Token 按钮
settings.html Gist 同步配置区 SHALL 提供"清除 Token"按钮，一键移除 sessionStorage 中的 Token 和 Gist ID。

#### Scenario: 清除 Token
- **WHEN** 用户点击"清除 Token"按钮
- **THEN** 调用 removeSecure('githubToken') + removeSecure('gistId')
- **AND** Token 输入框清空，Gist ID 显示"未同步"
- **AND** toast 提示"Token 已从本地清除"

### Requirement: 三路降级失败原因展示
fetchPricesNow 全部失败时 SHALL 显示具体失败原因。

#### Scenario: 三路全失败
- **WHEN** futsseapi、东财JSONP、新浪均失败
- **THEN** toast 显示"⚠ 行情获取失败：{失败原因}，已切换手动模式"
- **AND** 状态指示灯红色
- **AND** 失败原因示例："futsseapi 超时 / 东财无响应 / 新浪超时"

### Requirement: Gist 恢复冲突确认
restoreFromGist SHALL 在本地数据更新时弹出确认框。

#### Scenario: 本地更新覆盖确认
- **WHEN** 本地 state.lastBackup 比 Gist 数据的 lastBackup 更新
- **THEN** confirm() 弹窗"Gist 版本时间戳：X / 本地版本时间戳：Y / 本地数据更新，确认用 Gist 版本覆盖吗？"
- **AND** 用户确认后才恢复，取消则中止

#### Scenario: Gist 更新或无时间戳
- **WHEN** Gist 数据的 lastBackup >= 本地，或 Gist 无 lastBackup
- **THEN** 直接恢复不确认

### Requirement: 日志搜索清空按钮
journal.html 搜索过滤栏 SHALL 提供"× 清空"按钮。

#### Scenario: 一键清空筛选
- **WHEN** 用户点击"× 清空"按钮
- **THEN** 品种下拉重置为"全部"，类型下拉重置为"全部"，关键词输入框清空
- **AND** 重新渲染日志列表显示全部

### Requirement: 仪表盘盈亏曲线接真实数据
renderEquityChart SHALL 基于真实 equityHistory 和 closedTrades 数据绘制。

#### Scenario: 有 equityHistory 数据
- **WHEN** equityHistory 有 2 条以上记录
- **THEN** 绘制折线图，X 轴显示日期，Y 轴显示金额
- **AND** 起始资金画水平基线
- **AND** 当前净值标注末端点

#### Scenario: equityHistory 不足但 closedTrades 有数据
- **WHEN** equityHistory 不足 2 条但 closedTrades 有平仓记录
- **THEN** 用 closedTrades 按日期累加 pnl 构建临时曲线
- **AND** 从 initEquity 起始，每笔平仓追加一个节点

#### Scenario: 平仓时追加资金曲线节点
- **WHEN** 用户平仓一笔交易
- **THEN** updateEquityHistory 追加一个 {date: today, equity: 当前净值} 节点
- **AND** 若当天已有节点则更新而非追加

## MODIFIED Requirements

### Requirement: renderEquityChart 图表增强
renderEquityChart SHALL 增强：X 轴日期标签、Y 轴金额标签、起始资金基线、当前净值末端标注。当前仅有折线+渐变填充，无坐标轴标签。

### Requirement: updateEquityHistory 平仓触发
updateEquityHistory SHALL 在平仓时被调用（当前仅在 onPriceUpdate 中调用）。closePosition 函数末尾新增 updateEquityHistory() 调用。
