# Checklist

## P1: 清除 Token 按钮
- [ ] settings.html Gist 配置区新增"清除 Token"按钮
- [ ] 点击调用 removeSecure('githubToken') + removeSecure('gistId')
- [ ] Token 输入框清空，gistIdDisplay 显示"未同步"
- [ ] toast 提示"Token 已从本地清除"

## P2-1: 三路降级失败提示
- [ ] fetchPricesNow 新增 fetchFailReasons 数组
- [ ] 每路失败时 push 具体原因
- [ ] 全部失败 toast 显示具体原因拼接
- [ ] 状态指示灯红色

## P2-2: Gist 恢复冲突确认
- [ ] restoreFromGist 对比本地与 Gist 的 lastBackup 时间戳
- [ ] 本地更新时 confirm() 弹窗确认
- [ ] 取消则中止不恢复
- [ ] Gist 更新或无 lastBackup 时直接恢复

## P3-1: 日志搜索清空按钮
- [ ] journal.html 搜索栏右侧新增"× 清空"按钮
- [ ] 点击重置品种/类型/关键词三个过滤器
- [ ] 重新渲染显示全部日志

## P3-2: 仪表盘盈亏曲线
- [ ] closePosition 末尾调用 updateEquityHistory
- [ ] FTApp 导出 updateEquityHistory
- [ ] renderEquityChart X 轴日期标签
- [ ] renderEquityChart Y 轴金额标签
- [ ] renderEquityChart 起始资金基线（虚线）
- [ ] renderEquityChart 当前净值末端标注
- [ ] equityHistory 不足时用 closedTrades 构建临时曲线
- [ ] 临时曲线从 initEquity 起始累加 pnl

## 端到端验证
- [ ] node -c 语法检查 app-core.js 通过
- [ ] node -c 语法检查 ui-core.js 通过
- [ ] 推送所有修改文件到 GitHub 成功
