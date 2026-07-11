# Tasks

- [ ] Task 1: settings.html 新增清除 Token 按钮
  - [ ] SubTask 1.1: Gist 同步配置区"立即同步"/"从Gist恢复"按钮旁新增"清除 Token"按钮
  - [ ] SubTask 1.2: 点击调用 removeSecure('githubToken') + removeSecure('gistId')，清空输入框，gistIdDisplay 显示"未同步"，toast 提示

- [ ] Task 2: app-core.js fetchPricesNow 失败提示细化
  - [ ] SubTask 2.1: 新增 fetchFailReasons 数组，每路失败时 push 具体原因
  - [ ] SubTask 2.2: 全部失败分支 toast 显示"⚠ 行情获取失败：{原因拼接}，已切换手动模式"
  - [ ] SubTask 2.3: 状态指示灯改红色（setDataSourceStatus('offline', ...) 已是红色，确认即可）

- [ ] Task 3: app-core.js restoreFromGist 冲突确认
  - [ ] SubTask 3.1: 恢复前对比本地 state.lastBackup 与 Gist 数据的 lastBackup
  - [ ] SubTask 3.2: 本地更新时 confirm() 弹窗确认，取消则中止
  - [ ] SubTask 3.3: Gist 更新或无 lastBackup 时直接恢复

- [ ] Task 4: journal.html 搜索清空按钮
  - [ ] SubTask 4.1: 搜索过滤栏右侧新增"× 清空"按钮
  - [ ] SubTask 4.2: 点击重置三个过滤器并重新渲染

- [ ] Task 5: app-core.js updateEquityHistory 平仓触发
  - [ ] SubTask 5.1: ui-core.js closePosition 函数末尾新增 FTApp.updateEquityHistory() 调用
  - [ ] SubTask 5.2: FTApp 导出 updateEquityHistory

- [ ] Task 6: ui-core.js renderEquityChart 增强
  - [ ] SubTask 6.1: X 轴日期标签（首尾日期）
  - [ ] SubTask 6.2: Y 轴金额标签（最大/最小值，已有但优化位置）
  - [ ] SubTask 6.3: 起始资金水平基线（虚线）
  - [ ] SubTask 6.4: 当前净值末端标注点

- [ ] Task 7: ui-core.js renderDashboard 临时曲线兜底
  - [ ] SubTask 7.1: equityHistory 不足 2 条但 closedTrades 有数据时，用 closedTrades 构建临时曲线
  - [ ] SubTask 7.2: 从 initEquity 起始，按 closeTime 排序累加 pnl

- [ ] Task 8: 语法检查 + 推送 GitHub
  - [ ] SubTask 8.1: node -c 语法检查 app-core.js 和 ui-core.js
  - [ ] SubTask 8.2: 推送修改的文件到 GitHub

# Task Dependencies
- Task 5, 6, 7 涉及 ui-core.js 和 app-core.js 的资金曲线相关函数，建议顺序执行
- Task 1, 2, 3, 4 互相独立，可并行
- Task 8 依赖 Task 1-7 全部完成
