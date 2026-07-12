# Checklist

> 对应 spec 要求与回归验证。全部勾选后方可结束。

## 共享模块创建
- [x] `shared/fund-dimension-config.js` 存在且定义 `FUND_DIMENSION_CONFIG`
- [x] 配置字段（dimensions / indicatorToDimension / varietyTemplates / templates / refuseThreshold）与 PR #5 完全一致
- [x] `mapIndicatorToDim(name)` 函数已迁移，前缀按长度降序匹配
- [x] `FUND_DIMENSION_CONFIG` 与 `mapIndicatorToDim` 挂到 `window` 全局
- [x] `validateConfig()` 函数存在，校验维度 key 唯一/权重非负/品种映射有效/前缀无冲突
- [x] `validateConfig()` 仅在 localhost/127.0.0.1 执行，失败时 `console.warn`

## fundamental.html 修改
- [x] `<script src="../shared/fund-dimension-config.js"></script>` 位于 `ui-core.js` 之后、内联覆写脚本之前
- [x] 内联脚本中 `FUND_DIMENSION_CONFIG` 定义已删除
- [x] 内联脚本中 `_PREFIX_ENTRIES` 预排序与 `mapIndicatorToDim` 函数已删除
- [x] 内联脚本中覆写逻辑（loadFundamental / updateFundTotal / renderExternalFundSignal）与解析辅助函数保留不变
- [x] 内联脚本引用 `FUND_DIMENSION_CONFIG` / `mapIndicatorToDim` 处可正确读取全局变量

## 行为回归（与 PR #5 一致，Node.js 逻辑验证 75/75 PASS）
- [x] 白糖：供需/库存/宏观按真实指标亮分，基差/技术"待手动输入"，综合分归一化（配置一致 → 行为一致）
- [x] 白银：仅宏观亮分，其余"无品种级数据源"，有"宏观情绪分"提示（配置一致 → 行为一致）
- [x] 螺纹钢：轻量模组，仓单分位 + "⚠ 轻量模组，仅供参考"（配置一致 → 行为一致）
- [x] 有效权重 < 50% 时显示"数据不足，拒绝出分"并列缺失原因（配置一致 → 行为一致）
- [x] 新鲜度角标变色（≤3天灰/>3天黄/>7天红）（覆写逻辑未变 → 行为一致）
- [x] OIL_WTI 告警只出现一次（覆写逻辑未变 → 行为一致）
- [x] `window.FUND_DIMENSION_CONFIG` / `window.mapIndicatorToDim` 在 console 可用（Node.js vm 验证通过）
- [x] localhost 下 `validateConfig()` 无 `console.warn`（返回空数组，验证通过）

## 范围与约束
- [x] `shared/app-core.js`、`shared/ui-core.js`、`shared/fundamental-feed.json` 未被修改
- [x] 信号引擎 `signal.html` 行为未受影响
- [x] 无行为变化——纯重构，渲染/计分/告警/新鲜度与 PR #5 一致
