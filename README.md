# Codex Claude Bridge

讓 Codex 可以向本機的 Claude Code CLI 詢問第二意見、程式碼審查、對抗式審查，或請 Claude 提出「文字版 diff 建議」的 Codex skill。

這個專案的核心定位很簡單：

- Codex 是操作者，負責讀檔、改檔、執行命令與判斷結果。
- Claude 是顧問，只提供唯讀建議。
- Claude 不會直接修改你的檔案，也不會替你執行後續命令。

## 這個專案解決什麼問題

在開發時，你可能會希望 Codex 之外再有另一個模型幫忙看一次：

- 這個修法有沒有漏掉邊界情境？
- 目前的 git diff 是否有明顯 regression？
- 這個 migration、rollback 或資料安全設計有沒有風險？
- 是否能提出一個更小、更保守的 patch 方向？

`codex-claude-bridge` 提供一個薄薄的橋接層，讓 Codex 可以在需要時呼叫本機 `claude` CLI，並把目前的 git 狀態、staged diff、unstaged diff 等上下文整理給 Claude。Claude 的輸出會回到 Codex，由 Codex 再決定是否採納。

## 功能

- `setup`：檢查本機是否能找到 Claude Code CLI，並做一次 smoke test。
- `ask`：問 Claude 一個不需要 repo diff 的單次問題。
- `review`：請 Claude review 目前 git working tree。
- `adversarial-review`：請 Claude 從風險、邊界情境、rollback、資料遺失、可靠性等角度挑戰目前變更。
- `propose-diff`：請 Claude 只輸出建議的 unified diff 文字，Codex 需要自行審查後才可套用。

## 安全邊界

這個 bridge 預設是唯讀設計：

- 呼叫 Claude 時不授予編輯工具。
- 不要求 Claude 修改檔案。
- 不自動套用 Claude 建議的 patch。
- 不執行 Claude 建議的命令，除非 Codex 先審查並判斷可行。
- `review` 只會讀取 git status、diff stat、staged diff、unstaged diff 等 git 上下文。
- untracked files 只列出檔名，不直接讀取內容。
- 偵測到疑似 secret、token、password 等字樣時，會嘗試遮蔽明顯的 assignment。

Claude 的建議應被視為外部意見，而不是事實來源。最終決策仍由 Codex 和使用者共同確認。

## 需求

使用前請先準備：

- Codex，並啟用自訂 skills。
- Python 3。
- Git。
- Claude Code CLI，也就是本機可執行的 `claude` 指令。
- 已登入 Claude Code CLI。

你可以先確認本機是否有 `claude`：

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

如果 Codex 已經開著，安裝後建議重新開啟 Codex，或開一個新的 Codex session，讓 skill 清單重新載入。

### 使用 `.skill` bundle 安裝

此 repo 也包含已打包的 skill 檔：

```text
dist/codex-claude-bridge.skill
```

如果你的 Codex 環境支援從 `.skill` 匯入，可以直接使用這個檔案。你也可以手動解開到 skills 目錄：

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

成功時你會看到 Claude Code CLI 路徑、版本資訊，以及 smoke test 結果。

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

Codex 會依照 skill 描述判斷何時呼叫 bridge。Claude 回覆後，Codex 會整理、判斷並再回到你的工作流程。

## 直接執行 wrapper

你也可以不透過 Codex，直接在 skill 目錄執行 wrapper。

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

如果 Codex 裡找不到，但你的一般 terminal 找得到，通常是 GUI app 啟動時沒有繼承完整 shell PATH。可以重新開啟 Codex，或調整系統 shell 環境。

### `review` 顯示目前不是 git repo

`review`、`adversarial-review` 和 `propose-diff` 需要 git working tree。請切到專案 repo 後再執行。

如果只是想問一般問題，請使用 `ask`。

### Claude 的建議可以直接套用嗎？

不建議直接套用。這個專案刻意把 Claude 定位成顧問，因為任何模型都可能誤判上下文。請讓 Codex 或你自己先 review Claude 的建議，再決定是否採納。

## 專案結構

```text
.
├── SKILL.md
├── scripts/
│   └── claude_code_bridge.py
├── dist/
│   └── codex-claude-bridge.skill
└── README.md
```

## 授權

請見 [`LICENSE`](LICENSE)。
