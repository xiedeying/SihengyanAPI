# Sub2API 收割候选清单

> 来源：`Wei-Shaw/sub2api` `origin/main` 近 30 天 merged PR（412 个，初筛于 2026-09-14）。
> 远端同步点：fetch 于 2026-09-14（只读 `fetch --prune`，未动 `../sub2api` 工作树）。
> 状态机：待评估 → 已确认缺口/已有 → 已收割/忽略。点名后走单目标流程（等价判断→最小迁移→worker→reviewer）。

## 高级（建议优先看）

### 1. #7062 minimax 渠道监控白名单遗漏 — 待评估
- sub2api 改动：`channel_monitor_validate.go` +2 行，`monitorProviders` / `probeCapableProviders` 补 `MiniMax`（Fixes #7061）。
- Pixel 现状：架构不同，无这两个白名单 map，`validateProvider` 走 `providerAdapters` 注册表；且 `providerAdapters` 当前只有 openai/grok/anthropic/gemini 四项，无 minimax 条目。
- 初判：不是同一个 bug，但 Pixel 可能有“minimax 探活压根不支持”的对等缺口，需确认 minimax 在 Pixel 渠道监控里是否在支持范围内。

### 2. #5833 OpenAI Fast 缺 service_tier 强制 priority — 可能已有
- sub2api 改动：tier 缺失时强制 priority（`openai_fast_policy*`, `openai_gateway_forward.go` 等）。
- Pixel 现状：有 `openai_fast_policy_test.go`，默认策略覆盖 tier 缺失/别名场景。
- 初判：大概率已有，等价判断即可，大概率忽略。

### 3. #6890 OpenAI Images 大重构（direct 路径/用量/计费）— 待评估
- sub2api 改动：20 文件 +897/-130，新增 `openai_images_direct.go`（261 行）、direct payload/test，动用量与 `billing_service.go` / `pricing_service.go`。
- Pixel 现状：无 `openai_images_direct.go`、`image_output_accounting.go`；有 `openai_images*.go` 基础路径。
- 初判：Image 2.5 那次 Pixel 只收了模型与主控部分，这个 direct 大重构还没收；但体量大，需拆小点名（如只看用量计费口径）。

## 中级（按需点名）

### 4. #7057 Codex 最大 context window 保留 — 待评估
- sub2api 改动：`openai_codex_model_metadata.go`、`upstream_models.go`、前端 accounts API。
- Pixel 现状：无 `openai_codex_model_metadata.go` 文件。
- 初判：Pixel 模型元数据走别的路径，需确认是否有同类截断 bug。

### 5. #6929 Gemini 带内响应信号 — 待评估
- sub2api 改动：新增 `gemini_response_signal.go`，2xx 带内错误不再按上游失败登记。
- Pixel 现状：无 `gemini_response_signal.go`。
- 初判：大概率缺口，但需确认 Pixel Gemini 路径是否有对等错误归因逻辑。

### 6. #6974 DeepSeek V4.1 Flash 定价 — 待评估
- sub2api 改动：pricing json + billing 上下文 + 白名单。
- Pixel 现状：opencode 目录有 deepseek-v4-flash 系列，但 pricing 侧未确认 V4.1 条目。
- 初判：纯数据口径，大概率小补丁。

### 7. #7064/#7049/#7043 OpenAI WS 池容量 trio — 待评估
- sub2api 改动：同作者连续三刀，`openai_ws_pool.go` + config + 前端 ws 模式。
- Pixel 现状：有 `openai_ws_pool.go`。
- 初判：性能调优类，需 diff 对比三处是否已同步，不急。

## 忽略（Pixel 已有或无意义）

- #6874 Image 2.5 + OAuth 主控：仅剩 3 处小缺口（transform 环境变量透传、主控下线错误透出、Image 2.5 fallback 定价），见上次评估。
- 纯 UI/文案类（#7026 KeysView、#7053/#7054/#7055 ouchihao 系列表单刷新类）：Pixel 前端已分叉，收的意义不大。
