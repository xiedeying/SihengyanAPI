---
name: sub2api-harvest
description: Harvest a feature or fix from Wei-Shaw/sub2api into SihengyanAPI by native reimplementation. Use when user asks to sync, port, evaluate, or harvest a sub2api PR, commit, provider support, bugfix, or feature.
argument-hint: "<sub2api-PR|commit|feature>"
---

# Sub2API Harvest

将 `Wei-Shaw/sub2api` 的某个实现收割到 SihengyanAPI（origin 主仓库），以上游 `PIXEL-API/PixelAPI` 架构为准做原生重实现。

## Repo 定位（禁区）

- SihengyanAPI：主仓库，主产品，唯一交付目标。
- `PIXEL-API/PixelAPI`：真正的 upstream，只做正常 fetch/rebase，不经本 skill 处理。
- `Wei-Shaw/sub2api`：纯参考实现，默认路径 `${COMMANDCODE_PROJECT_DIR}/../sub2api`（可用参数覆盖）。
- 禁止：cherry-pick / merge sub2api 历史、不经重实现的整块复制、对 sub2api 建立运行时依赖、另建 Sidecar 或平行系统。

## 流程（$ARGUMENTS 为本次收割目标：发现 / PR 号 / commit / 功能描述）

0. 先 `git -C <sub2api-path> fetch origin --prune` 拿全量（只更新远端 refs，不碰工作树；禁止 `pull` / `rebase` / `checkout` / `merge`）。之后所有定位基于 `origin/main` 或指定 commit / PR head ref，用 `log` / `show` / `diff` 只读。
A. 发现模式（$ARGUMENTS 为空，或为“发现/最新/列表”类词时进入，不做迁移）：
   - 先读 `references/candidates.md`（候选清单证据，含远端同步点与状态机）；远端有新进展才增量追加，不重写已有评估。
   - 取 `origin/main` 最近 30 天 merged PR，按“影响面 × 是否已在 SihengyanAPI 存在”分级：严重 高级 中级 忽略。
   - 只做 notes 级初筛（标题/描述/改动文件清单），不读实现细节，不 fetch 之外的动作。
   - 输出清单：PR 号、标题、分类、严重度、SihengyanAPI 是否已有，候用户点名后再走下面单目标流程。
B. 单目标流程：
1. 在 sub2api 参考仓库定位对应实现：如果是 PR，列出全部 commits 全量评估；如果是单个 commit，先查归属 PR，有关联则把同 PR 其他 commits 一并纳入，避免只收一半。再定位改了哪些文件、核心逻辑、依赖的数据结构。
2. 分类（只选其一）：bugfix / provider 支持 / 调度功能 / 计费功能 / UI / 基础设施。
3. 检查 SihengyanAPI 是否已有等价能力：有则直接报告等价位置并停止，不做迁移。
4. 按 Pixel/SihengyanAPI 当前模块边界找到对应模块（账号、调度、共享、计费、账本、支付、管理），沿现有调用链落点。
5. 设计最小迁移方案：只覆盖本次目标，用本项目自己的数据结构和代码风格重写，不机械同步 sub2api 结构。
6. 实现按项目规则第 16 条走：分析映射由主 Agent 完成，代码修改优先委派给 `worker`；明显单点小改可由主 Agent 直接完成。
7. 验证默认只跑已有构建/测试做最小验证；仅当本次收割请求明确要求时才新增测试 case。
8. 实现后委派 `final-reviewer` 验收；BLOCK 时只修阻断项并复审一次。
9. PASS 即停止；复审仍不 PASS 则停止并报告阻断和现有成果，不循环。