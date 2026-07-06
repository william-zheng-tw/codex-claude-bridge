# Codex Claude Bridge

[![GitHub release](https://img.shields.io/github/v/release/william-zheng-tw/codex-claude-bridge?display_name=tag)](https://github.com/william-zheng-tw/codex-claude-bridge/releases)
[![License](https://img.shields.io/github/license/william-zheng-tw/codex-claude-bridge)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/william-zheng-tw/codex-claude-bridge?style=social)](https://github.com/william-zheng-tw/codex-claude-bridge/stargazers)

Language: [English](#english) | [繁體中文](#繁體中文)

## English

Codex Claude Bridge is a Codex skill that lets Codex ask the local Claude Code CLI for a read-only second opinion, code review, adversarial review, or proposed diff.

If this helps your Codex workflow, starring the repository helps other developers discover it.

Helpful links:

- [Use cases](docs/USE_CASES.md)
- [Latest release](https://github.com/william-zheng-tw/codex-claude-bridge/releases/latest)

The project has one clear boundary:

- Codex is the operator. It reads files, edits files, runs commands, and decides what to do.
- Claude is the advisor. It provides read-only feedback.
- Claude does not directly modify your files or run follow-up commands.

## Why This Exists

Codex is great at operating inside a repository, but sometimes you want a second model to challenge a change before Codex applies it. This skill makes that review loop lightweight: Codex can ask local Claude Code for focused feedback while keeping Claude read-only.

Use it when you want another perspective on correctness, edge cases, rollback risk, data-loss risk, or whether a proposed patch can be smaller.

## Copy-Paste Install

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/william-zheng-tw/codex-claude-bridge.git ~/.codex/skills/codex-claude-bridge
cd ~/.codex/skills/codex-claude-bridge
python3 scripts/claude_code_bridge.py setup
```

Restart Codex or open a new Codex session after installing.

## What This Solves

Sometimes you want another model to challenge Codex's work before a change is committed or shipped:

- Did this fix miss an edge case?
- Does the current git diff introduce a regression?
- Are there rollback, migration, concurrency, reliability, or data-loss risks?
- Is there a smaller and safer patch?

`codex-claude-bridge` gives Codex a thin bridge to the local `claude` CLI. For review-style commands, the wrapper collects git status, diff stats, staged diffs, and unstaged diffs, then sends that context to Claude. Claude's response comes back to Codex, and Codex decides whether the advice is correct and useful.

## Features

- `setup`: Check whether the local Claude Code CLI is available and run a smoke test.
- `ask`: Ask Claude a focused question that does not require repository diff context.
- `review`: Ask Claude to review the current git working tree.
- `adversarial-review`: Ask Claude to challenge the change from risk, rollback, data-loss, concurrency, and reliability angles.
- `propose-diff`: Ask Claude to return a proposed unified diff as text only. Codex must review it before applying anything.

## Safety Model

This bridge is intentionally read-only:

- It does not grant Claude editing tools.
- It does not ask Claude to modify files.
- It does not automatically apply Claude's patch suggestions.
- It does not execute commands suggested by Claude unless Codex reviews them first.
- `review` reads git status, diff stats, staged diffs, and unstaged diffs.
- Untracked files are listed by name only; their contents are not read.
- Secret-like assignments are redacted when detected.

Treat Claude's output as external advice, not ground truth.

## Requirements

Before using this skill, make sure you have:

- Codex with custom skills enabled.
- Python 3.
- Git.
- Claude Code CLI available as the local `claude` command.
- A signed-in Claude Code CLI session.

Check whether `claude` is available:

```bash
command -v claude
claude --version
```

## Installation

The recommended installation method is to clone this repository into your Codex skills directory:

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/william-zheng-tw/codex-claude-bridge.git ~/.codex/skills/codex-claude-bridge
```

After installation, restart Codex or open a new Codex session so the skill list can reload.

### Install From the `.skill` Bundle

This repository also includes a packaged skill file:

```text
dist/codex-claude-bridge.skill
```

You can download it from the [latest release](https://github.com/william-zheng-tw/codex-claude-bridge/releases/latest). If your Codex environment supports importing `.skill` files, use that file directly. You can also unzip it manually:

```bash
mkdir -p ~/.codex/skills
unzip dist/codex-claude-bridge.skill -d ~/.codex/skills
```

After unzipping, you should have:

```text
~/.codex/skills/codex-claude-bridge/SKILL.md
~/.codex/skills/codex-claude-bridge/scripts/claude_code_bridge.py
```

## Updating

If you installed with `git clone`, update with:

```bash
cd ~/.codex/skills/codex-claude-bridge
git pull --ff-only
```

Then restart Codex or open a new Codex session.

If you installed from the `.skill` bundle, download the newer bundle and unzip it into `~/.codex/skills` again, replacing the older copy.

## First Run

Run the setup check:

```bash
cd ~/.codex/skills/codex-claude-bridge
python3 scripts/claude_code_bridge.py setup
```

A successful setup prints the Claude Code CLI path, version information, and smoke test result.

If setup fails, check that:

- `claude` is on `PATH`.
- Claude Code CLI is installed.
- Claude Code CLI is signed in.

## Using It in Codex

After installation, ask Codex naturally:

```text
Ask Claude Code to review my current changes.
```

```text
Use codex-claude-bridge to challenge this migration plan.
```

```text
Ask Claude whether there is a smaller patch for this fix.
```

Codex will decide when to call the bridge based on the skill description. After Claude responds, Codex should evaluate the advice before acting on it.

## Running the Wrapper Directly

You can also run the wrapper yourself.

### Setup Check

```bash
python3 scripts/claude_code_bridge.py setup
```

### Ask a Focused Question

```bash
python3 scripts/claude_code_bridge.py ask "Challenge this implementation plan."
```

### Review the Current Git Diff

Run this inside a git repository:

```bash
python3 ~/.codex/skills/codex-claude-bridge/scripts/claude_code_bridge.py review
```

### Adversarial Review

```bash
python3 ~/.codex/skills/codex-claude-bridge/scripts/claude_code_bridge.py adversarial-review "Focus on rollback and data-loss risks."
```

### Ask for a Proposed Diff

```bash
python3 ~/.codex/skills/codex-claude-bridge/scripts/claude_code_bridge.py propose-diff "Suggest the smallest safe fix."
```

`propose-diff` asks Claude to return a text-only unified diff. Review it before applying anything.

## FAQ

### `claude` cannot be found

Make sure Claude Code CLI is installed and available on `PATH`:

```bash
command -v claude
```

If your regular terminal can find `claude` but Codex cannot, the Codex app may not have inherited your full shell environment. Restart Codex or adjust your shell environment.

### `review` says the current directory is not a git repository

`review`, `adversarial-review`, and `propose-diff` must run inside a git working tree.

Use `ask` for general questions that do not need git context.

### Can I apply Claude's suggested patch directly?

You should review it first. This project deliberately treats Claude as an advisor because any model can misunderstand the context. Let Codex or a human review the suggestion before applying it.

## Community

- Open an issue for bugs or installation problems.
- Start a GitHub Discussion for questions, workflow ideas, or usage stories.
- Share the repo with a concrete use case, such as "Codex asks Claude for a read-only adversarial review before I apply a patch."

## Project Structure

```text
.
├── .github/
│   └── ISSUE_TEMPLATE/
├── assets/
│   └── social-preview.png
├── CONTRIBUTING.md
├── docs/
│   └── USE_CASES.md
├── SKILL.md
├── scripts/
│   └── claude_code_bridge.py
├── dist/
│   └── codex-claude-bridge.skill
└── README.md
```

## License

See [`LICENSE`](LICENSE).

---

## 繁體中文

Codex Claude Bridge 是一個 Codex skill，讓 Codex 可以向本機的 Claude Code CLI 詢問唯讀的第二意見、程式碼審查、對抗式審查，或請 Claude 提出 diff 建議。

如果這個工具有幫到你的 Codex workflow，幫 repo 按 star 可以讓更多開發者發現它。

相關連結：

- [使用情境](docs/USE_CASES.md)
- [Latest release](https://github.com/william-zheng-tw/codex-claude-bridge/releases/latest)

這個專案的邊界很清楚：

- Codex 是操作者，負責讀檔、改檔、執行命令與判斷結果。
- Claude 是顧問，只提供唯讀建議。
- Claude 不會直接修改你的檔案，也不會替你執行後續命令。

## 為什麼需要這個工具

Codex 很適合在 repo 裡執行操作，但有時候你會希望另一個模型在套用變更前先挑戰一次。這個 skill 讓 Codex 可以輕量地詢問本機 Claude Code，同時保持 Claude 唯讀。

適合用在你想多看一次 correctness、edge cases、rollback 風險、資料遺失風險，或想確認 patch 是否能更小的時候。

## 複製貼上安裝

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/william-zheng-tw/codex-claude-bridge.git ~/.codex/skills/codex-claude-bridge
cd ~/.codex/skills/codex-claude-bridge
python3 scripts/claude_code_bridge.py setup
```

安裝後請重新開啟 Codex，或開一個新的 Codex session。

## 這個專案解決什麼問題

有時候你會希望 Codex 之外再有另一個模型幫忙檢查：

- 這個修法有沒有漏掉邊界情境？
- 目前的 git diff 是否引入 regression？
- migration、rollback、concurrency、可靠性或資料遺失風險是否被低估？
- 是否有更小、更安全的 patch？

`codex-claude-bridge` 提供一個薄薄的橋接層，讓 Codex 可以呼叫本機 `claude` CLI。對於 review 類指令，wrapper 會整理 git status、diff stat、staged diff、unstaged diff 等上下文給 Claude。Claude 的輸出會回到 Codex，由 Codex 再判斷是否正確、是否值得採納。

## 功能

- `setup`：檢查本機是否能找到 Claude Code CLI，並做一次 smoke test。
- `ask`：問 Claude 一個不需要 repo diff 的單次問題。
- `review`：請 Claude review 目前 git working tree。
- `adversarial-review`：請 Claude 從風險、rollback、資料遺失、concurrency、可靠性等角度挑戰目前變更。
- `propose-diff`：請 Claude 只輸出文字版 unified diff，Codex 需要審查後才可套用。

## 安全模型

這個 bridge 預設是唯讀設計：

- 不授予 Claude 編輯工具。
- 不要求 Claude 修改檔案。
- 不自動套用 Claude 建議的 patch。
- 不執行 Claude 建議的命令，除非 Codex 先審查。
- `review` 只會讀取 git status、diff stat、staged diff、unstaged diff。
- untracked files 只列出檔名，不直接讀取內容。
- 偵測到疑似 secret assignment 時會嘗試遮蔽。

Claude 的建議應被視為外部意見，而不是事實來源。

## 需求

使用前請先準備：

- Codex，並啟用自訂 skills。
- Python 3。
- Git。
- Claude Code CLI，也就是本機可執行的 `claude` 指令。
- 已登入 Claude Code CLI。

確認本機是否有 `claude`：

```bash
command -v claude
claude --version
```

## 安裝

建議直接把這個 repo clone 到 Codex 的 skills 目錄：

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/william-zheng-tw/codex-claude-bridge.git ~/.codex/skills/codex-claude-bridge
```

安裝後建議重新開啟 Codex，或開一個新的 Codex session，讓 skill 清單重新載入。

### 使用 `.skill` bundle 安裝

此 repo 也包含已打包的 skill 檔：

```text
dist/codex-claude-bridge.skill
```

你可以從 [latest release](https://github.com/william-zheng-tw/codex-claude-bridge/releases/latest) 下載。如果你的 Codex 環境支援從 `.skill` 匯入，可以直接使用這個檔案。你也可以手動解開到 skills 目錄：

```bash
mkdir -p ~/.codex/skills
unzip dist/codex-claude-bridge.skill -d ~/.codex/skills
```

解開後應該會得到：

```text
~/.codex/skills/codex-claude-bridge/SKILL.md
~/.codex/skills/codex-claude-bridge/scripts/claude_code_bridge.py
```

## 更新

如果你是用 `git clone` 安裝，更新方式是：

```bash
cd ~/.codex/skills/codex-claude-bridge
git pull --ff-only
```

更新後同樣建議重新開啟 Codex 或開一個新的 Codex session。

如果你是用 `.skill` bundle 安裝，請下載新版 bundle，重新解壓到 `~/.codex/skills`，覆蓋舊版 skill。

## 第一次使用

先執行 setup 檢查：

```bash
cd ~/.codex/skills/codex-claude-bridge
python3 scripts/claude_code_bridge.py setup
```

成功時會看到 Claude Code CLI 路徑、版本資訊，以及 smoke test 結果。

如果失敗，請確認：

- `claude` 是否在 `PATH` 上。
- Claude Code CLI 是否已安裝。
- Claude Code CLI 是否已登入。

## 在 Codex 裡使用

安裝完成後，你可以直接用自然語言請 Codex 啟用這個 skill。例如：

```text
請用 Claude Code 幫我 review 目前改動。
```

```text
請用 codex-claude-bridge 讓 Claude 挑戰這個 migration plan。
```

```text
請問 Claude 對這個修法有沒有更小的 patch 建議。
```

Codex 會依照 skill 描述判斷何時呼叫 bridge。Claude 回覆後，Codex 應先評估建議，再決定是否採納。

## 直接執行 wrapper

你也可以不透過 Codex，直接執行 wrapper。

### 檢查環境

```bash
python3 scripts/claude_code_bridge.py setup
```

### 問一個單次問題

```bash
python3 scripts/claude_code_bridge.py ask "Challenge this implementation plan."
```

### Review 目前 git diff

請在 git repo 裡執行：

```bash
python3 ~/.codex/skills/codex-claude-bridge/scripts/claude_code_bridge.py review
```

### 對抗式 review

```bash
python3 ~/.codex/skills/codex-claude-bridge/scripts/claude_code_bridge.py adversarial-review "Focus on rollback and data-loss risks."
```

### 請 Claude 提出 diff 建議

```bash
python3 ~/.codex/skills/codex-claude-bridge/scripts/claude_code_bridge.py propose-diff "Suggest the smallest safe fix."
```

`propose-diff` 只會要求 Claude 輸出文字版 unified diff。請先審查內容，再由 Codex 或你自己決定是否套用。

## 常見問題

### 找不到 `claude`

請確認 Claude Code CLI 已安裝，且 `claude` 在 shell 的 `PATH` 上：

```bash
command -v claude
```

如果 Codex 裡找不到，但你的一般 terminal 找得到，通常是 GUI app 啟動時沒有繼承完整 shell 環境。可以重新開啟 Codex，或調整系統 shell 環境。

### `review` 顯示目前不是 git repo

`review`、`adversarial-review` 和 `propose-diff` 需要 git working tree。請切到專案 repo 後再執行。

如果只是想問一般問題，請使用 `ask`。

### Claude 的建議可以直接套用嗎？

不建議直接套用。這個專案刻意把 Claude 定位成顧問，因為任何模型都可能誤判上下文。請讓 Codex 或你自己先 review Claude 的建議，再決定是否採納。

## 社群

- bug 或安裝問題可以開 issue。
- 使用問題、workflow 想法、實際案例可以開 GitHub Discussion。
- 分享時建議帶具體使用情境，例如「Codex 套 patch 前，先請 Claude 做唯讀對抗式 review」。

## 專案結構

```text
.
├── .github/
│   └── ISSUE_TEMPLATE/
├── assets/
│   └── social-preview.png
├── CONTRIBUTING.md
├── docs/
│   └── USE_CASES.md
├── SKILL.md
├── scripts/
│   └── claude_code_bridge.py
├── dist/
│   └── codex-claude-bridge.skill
└── README.md
```

## 授權

請見 [`LICENSE`](LICENSE)。
