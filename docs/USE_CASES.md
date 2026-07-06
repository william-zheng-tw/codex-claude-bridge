# Use Cases / 使用情境

Language: [English](#english) | [繁體中文](#繁體中文)

## English

Codex Claude Bridge is useful when you want Codex to stay in control, but you still want another model to challenge the work.

## 1. Review Before Applying a Patch

Ask Codex:

```text
Use Claude Code to review the current diff before applying the patch.
```

Good for:

- Catching obvious regressions.
- Checking missing tests.
- Getting a second read on edge cases.

## 2. Adversarial Review for Risky Changes

Ask Codex:

```text
Use codex-claude-bridge for an adversarial review focused on rollback, concurrency, and data-loss risks.
```

Good for:

- Migrations.
- Auth or permission changes.
- Data deletion or overwrite logic.
- Background jobs and retries.

## 3. Proposed Diff Without Write Access

Ask Codex:

```text
Ask Claude for the smallest proposed diff, but do not apply it automatically.
```

Good for:

- Comparing implementation strategies.
- Finding a smaller fix.
- Getting patch-shaped feedback while preserving Codex review.

## 4. Architecture or Plan Critique

Ask Codex:

```text
Ask Claude to challenge this implementation plan before we edit files.
```

Good for:

- Picking between two approaches.
- Checking assumptions before writing code.
- Stress-testing a rollout plan.

## 繁體中文

當你希望 Codex 保持操作權，但又想讓另一個模型挑戰目前工作時，Codex Claude Bridge 會很有用。

## 1. 套 patch 前先 review

可以請 Codex：

```text
請用 Claude Code 在套用 patch 前 review 目前 diff。
```

適合：

- 抓明顯 regression。
- 檢查缺少測試的地方。
- 多看一次 edge cases。

## 2. 高風險變更的對抗式 review

可以請 Codex：

```text
請用 codex-claude-bridge 做 adversarial review，聚焦 rollback、concurrency 和 data-loss 風險。
```

適合：

- migrations。
- auth 或 permission 變更。
- 資料刪除或覆寫邏輯。
- background jobs 與 retry 機制。

## 3. 只請 Claude 提 diff，不給寫入權

可以請 Codex：

```text
請 Claude 提出最小 proposed diff，但不要自動套用。
```

適合：

- 比較不同實作策略。
- 尋找更小的修法。
- 取得 patch 形狀的建議，但仍由 Codex 審查。

## 4. 架構或計畫挑戰

可以請 Codex：

```text
請 Claude 在我們改檔前挑戰這個 implementation plan。
```

適合：

- 在兩個做法間選擇。
- 寫 code 前檢查假設。
- 壓力測試 rollout plan。
