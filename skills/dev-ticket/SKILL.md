---
name: dev-ticket
description: 開發 bot（Dev A／B／C 或任何接開發票的 bot）做一張 GitHub 開發票的完整流程：接票檢查、worktree 與分支、用 Codex／Antigravity 實作的正確指令、Mac／iOS 測試交給 MBA 測試員、開 draft PR、回報格式、合併後清理。接到 repo#issue 要寫 code、開 PR 時使用。中文關鍵字：接票、開發票、開 PR、worktree、交件、回報 Paula。
---

# dev-ticket：做一張開發票

一次只做一張票。誰派的票就回報給誰（多半是專案經理 Paula）。

## 0. 主對話保持小（省 token，Ray 11-09）

Codync 沒有「開新對話」，你的對話會一路累積，之後每一步都要重讀整段。11-02～11-09 三個 Dev 每步平均重讀 48 萬 token，佔全部用量七成，其中約三分之一花在 sleep 迴圈和反覆查 gh 狀態。所以不管派工訊息有沒有寫，都照這幾條做：

- **整張票一定交給一個 subagent**（Agent 工具）。這是必做，不是建議。收到派工後，主對話的第一個動作就是開 subagent：把派工原文和這份 skill 的路徑交給它，由它做第 1～8 步。主對話只做三件事：派 subagent、收結論、回報。不在主對話讀 log、跑測試、看 diff、下 `gh` 或 `git` 指令，也不等待。
  - 續做、修 PR 意見、驗證別人的 PR 也一樣：每次開一個新的 subagent，把 PR 連結和現況交給它，不要叫它翻之前的對話。
  - 11-09～11-12 的紀錄：Dev C 在主對話自己下了 1,239 次 Bash，對話兩天被壓縮 58 次，每次壓縮都會丟細節。mars-dawn#683、website#366 的 PR 內文寫著「全部自己做，沒有委派」。
  - **subagent 用哪一級，照 pilotfish 的分配**（Ray 11-12），派工單有寫就照派工單：
    - 查找、讀檔：`pilotfish:scout`（Haiku）
    - 機械式工作（照貼文案、merge main、PR 收尾、照樣補測試）：`pilotfish:mech-executor`（Sonnet）
    - 一般實作、修 bug：`pilotfish:executor`（Sonnet）
    - 驗證：`pilotfish:verifier`（Opus）；安全相關：`pilotfish:security-reviewer`、`pilotfish:security-executor`（Opus）
    - 實作要用 Opus，只限派工單寫明原因的票（架構、跨檔大改、根因不明）。派工單只寫「Claude」時用 `pilotfish:executor`，不要自己把 subagent 的 model 設成 opus。11-12 當天 Dev A 的 subagent 有 83% 跑 Opus，花費是前幾天的 5 倍。
    - PR 內文的「subagent」欄寫用了哪一級。
  - 開不了 subagent（工具不存在或報錯）才自己做，並在 PR 內文和回報寫明原因。
- **指令輸出要短**（Ray 11-12）。Dev 的對話內容九成以上是 Bash 輸出。
  - 看 PR 狀態用 `pr-status <repo>#<n>`（head SHA、draft、CI 各格、最後 3 則留言、鎖），只印十幾行。只看鎖用 `pr-status --locks`。
  - 其他 `gh` 指令一律帶 `--json <欄位> --jq`，不要整段印 `gh pr view`、`gh pr checks`、`gh issue view --comments`。
  - build log、測試輸出寫到檔案，只看結尾：`<指令> > run.log 2>&1; tail -n 30 run.log`。要找錯誤就 `grep -n -m 20 'error' run.log`。
- **不輪詢**。不用 `sleep`／`until`／`while` 迴圈等 process 或鎖，也不反覆跑 `gh pr checks`、`gh pr view` 看狀態有沒有變。
  - 等 Codex／agy：前景一個指令跑到結束（Bash timeout 設到 600000，最長 10 分鐘）。會超過 10 分鐘就用 Bash 的背景執行，等完成通知，中間不要去查。
  - 等 MBA 鎖、CI、測試員、Ray：照規則排隊或留言，然後回報「等待中：<等什麼>，<連結>」並結束這一輪。派工的人會再叫你。
- **回報短**：第 5 步的固定欄位就夠，log 和 diff 放 PR，回報只給連結。

## 1. 接票檢查

交辦要有這五項，缺一項就回問派工的人，不要猜：
- repo#issue
- 目標
- 範圍（改哪些、不改哪些）
- 驗收條件
- 實作模型（Claude／Codex／Antigravity）

再到 issue 上確認同樣的內容寫在那裡。issue 上沒寫的決定，不要自己定，見下面「要 Ray 決定時」。

## 2. Worktree 與分支

- Repo 在 `~/Projects/<語言>/<repo>`，gh 已登入 redtear1115。
- **先查有沒有撞票**（這週撞過 3 次）：`gh pr list -R <repo> --search "<issue號>" --state open` 和 `git worktree list | grep <issue號>`。已經有人在做就停下來回報派工的人，不要開第二份。
- 一律開 git worktree、自己的分支，分支名帶 issue 號和自己的代號：`feat/<issue>-<短名>`、`fix/<issue>-<短名>`，worktree 資料夾名加 `-deva`／`-devb`／`-devc`。
- JS 專案建好 worktree 後跑 `npm ci`。不要 symlink 主 checkout 的 node_modules：舊的 install 缺套件，會讓 tsc 和整批 suite 假性失敗（這週 4 次）。
- 不碰別人的分支、worktree、PR。
- 先讀 repo 的 CLAUDE.md／AGENTS.md。
- 要把 main 合進自己的 PR 分支時，用 merge commit（`git merge origin/main`），不要 rebase 已經推出去的分支。CHANGELOG 衝突時兩邊的條目都留在 `[Unreleased]` 底下。

## 3. 實作模型

有指定就照指定，沒指定就自己做（Claude）。外部 CLI 照第 0 步的方式等它結束（不用迴圈輪詢），跑完確認 process 已經結束：

```bash
# Codex（ChatGPT Plus）
model-run --label "<repo>#<n> <做什麼>" --role 實作 -- \
  codex exec -C <worktree> -s workspace-write -o <結果檔> "<任務>" </dev/null

# Antigravity：Gemini 4 Argon 或 Claude Opus 5.5，一定要用 --model 寫明
model-run --label "<repo>#<n> <做什麼>" --role 實作 -- \
  agy -p "<任務>" --model gemini-4-argon-high --add-dir <worktree> --mode accept-edits
model-run --label "<repo>#<n> <做什麼>" --role 實作 -- \
  agy -p "<任務>" --model claude-opus-5-5-high --add-dir <worktree> --mode accept-edits

# claude-auto（Claude API 贈金）：要用 --model 寫明，工具清單用等號加逗號
claude-auto --label "<repo>#<n> <做什麼>" --model claude-opus-5-5 --allowedTools=Read,Edit,Bash "<任務>"
```

- **agy 和 codex 一律用 `model-run` 包起來跑**（Ray 11-12）。它不改指令的行為，只在 `~/.config/model-runs/runs.jsonl` 多記一行：工具、模型、秒數、exit、有沒有撞額度。`--role` 寫 `實作` 或 `驗證`。`model-run --summary` 看彙整。claude-auto 有自己的帳本，不用包。
  - 撞額度和 Codex token 數是用字樣比對抓的，還沒對過真的輸出。跑完看一眼最後一筆紀錄（`tail -n 1 ~/.config/model-runs/runs.jsonl`），不對就在回報裡寫。
- **claude-auto**：沒帶 `--model` 會用預設模型，不一定是派工單要的那個。「實作模型」填帳本裡記的實際 model id（`~/.config/claude-auto/usage.json` 的 `model`），不要只填「claude-auto」。工具清單寫 `--allowedTools=A,B`；寫成 `--allowedTools A B "<任務>"` 時，任務會被當成工具名吃掉。

- **模型要寫完整 id**：agy 裡同時有 Gemini 4 Argon 和 Claude Opus 5.5，評分時分開算。PR 和回報的「實作模型」寫 `<CLI>／<model id>`，例如 `agy／gemini-4-argon-high`、`agy／claude-opus-5-5-high`、`codex／<model>`、`Claude Code／<model>`，不要只寫「agy」或「Gemini」。中途換了模型，就把每個模型做了哪一段都寫出來。
- 可用的 model id 用 `agy models` 查。

### agy 無頭模式（-p）怎麼跑才不會被擋

沒給 `--settings` 時，第一個 shell 指令就會被拒；`-p` 模式只要有一個指令被拒，整個 run 就停掉，輸出「jetski: no output produced … auto-denied」，一個檔都不會改。website#347 前 3 次都是這樣，下面是第 4 次成功的寫法（11-02，Dev A）。

1. 在 worktree 放 `settings.json`，只開唯讀指令，`<worktree>` 換成完整路徑：

   ```json
   {"permissions":{"allow":["command(grep)","command(rg)","command(sed -n)","command(cat)","command(ls)","command(wc)","command(head)","command(tail)","command(find)","command(git diff)","command(git status)","command(sips)"]},"trustedWorkspaces":["<worktree>"]}
   ```

   - 寫法是 `command(程式名)`，可以帶開頭的參數，例如 `command(sed -n)`。**不能加萬用字元**，`command(grep *)` 會被拒。
   - `trustedWorkspaces` 是照全域設定加的，還沒單獨測過拿掉會怎樣。
2. 任務說明（`brief.md`）結尾一定要加這段原文：

   ```
   Tool rules for this run: only single, read-only commands (grep, cat, sed -n, ls, head, tail, wc, find, sips); no loops, pipes, `;`, `&&` or subshells, or they will be denied and the run ends. Edit files with your edit tool, not shell commands.
   ```

   複合指令（迴圈、管線、`;`、`&&`、子 shell）整段會被拒，就算裡面每個程式都在允許清單上也一樣。它會想查的事實（尺寸、常數、檔案位置）先寫進任務說明，讓它少跑指令。
3. 在 worktree 裡跑：

   ```bash
   model-run --label "<repo>#<n> <做什麼>" --role 實作 -- \
     agy -p "$(cat brief.md)" --model <model id> --add-dir <worktree> --mode accept-edits --settings settings.json </dev/null > agy.log 2>&1
   ```

4. 卡住時，看 `~/.gemini/antigravity-cli/brain/` 裡最新的資料夾，找 `.system_generated/logs/transcript.jsonl`，搜尋 `run_command`，看是哪個指令被拒。
5. `settings.json`、`brief.md`、`agy.log` 不要 commit。
6. 不要用 `--dangerously-skip-permissions`。
7. agy 不會印出 token 數，回報的額度欄寫「agy 沒有回報」。看到 `RESOURCE_EXHAUSTED (429)` 就是額度用完了：Gemini 4 Argon 的 low、medium、high 共用同一份額度，換檔位沒用。
- Codex 一定要加 `</dev/null`，不然會等 stdin 卡住。macOS 沒有 `timeout` 指令，要限時就用 Bash 工具本身的 timeout；超過 10 分鐘的用背景執行等完成通知，不要寫迴圈去查。
- 外部模型改完，自己看 diff，不要直接相信。
- 額度用完或沒登入：照 agy → Codex → Claude 的順序往下換，回報裡寫明換了什麼、為什麼。**模型試用期間例外**：派工單指定了模型，就不要自己換，停下來回報派工的人，由派工的人決定。不要替 Ray 登入。細節見 vault `基礎設施/AI 訂閱與 CLI.md`。

## 4. 測試

- **Mac／iOS 專案**（mars-dawn、mars-dawn-kit、dq-island、charspin 等）：build、測試、模擬器、實機和 archive 一律交給 MBA 測試員。不要自己 ssh 到 MBA，也不要在 Mini 上跑模擬器或會搶畫面的測試。例外（Ray 11-07）：Paula 交辦時，mars-dawn 的 `--part offscreen` 測試和反向檢查可在 Mini 跑，要先拿 `SCREEN.md` 的鎖；kit 的 SwiftPM 測試也可以。走查、錄影、screen 測試仍在 MBA。照 `mba-test-request` skill 寫測試需求，交給派工的人。
- **非 Apple 專案**（網站、Python 等）：在自己的 worktree 跑。
- 遇到 macOS 權限視窗、要密碼、要登入、要簽章：停下來回報，不要空等。
- 等 CI 或測試員結果：不要反覆查。回報「等待中」結束這一輪，結果出來派工的人會再叫你（見第 0 步）。

## 5. 交件

1. 推分支，開 **draft PR** 連回 issue（`Closes #n` 或 `Refs #n`）。
2. PR 內文：
   ```
   ## 改了什麼
   ## 怎麼測的
   ## 實作模型
   ## subagent（有／沒有；沒有就寫原因）
   ## 驗證模型（由派工的人另派，先留空）
   ## 測試結果（測試員的結果貼在 PR 留言，這裡放連結）
   ## 進度（做到哪、剩什麼；每次續做就覆寫）

   Co-authored-by: Claude <<bot-email>>
   ```
   - **不另開 issue**（Ray 11-11）：子步驟、後續修改、發現的小問題都記在這張 PR 的「進度」段，或寫成同一張票的子任務清單。只有要 Ray 決定的事、要交給別的 bot 的事、或範圍外的新 bug，才開新 issue；開之前先問派工的人。
   - **重大設計決定**：在同一個 PR 加一份 `docs/` 的決策檔（例如 mars-dawn 的 `docs/no-in-app-data-decision.md`），寫決定、原因、放棄的選項。小決定寫在「改了什麼」就好。
3. 不自己驗證自己的 PR。驗證的人照 `pr-verify` skill 做。
4. **回報前自查**：說「通過」「CI 綠」「已寫好」之前，重讀原始結果（測試輸出的結尾、`pr-status <repo>#<n>`），不要照記憶或被截斷的輸出寫。
5. 回報給派工的人，固定欄位：
   ```
   PR：<URL>
   head：<40 碼 SHA>
   CI：<run URL>（綠／紅／跑中）
   測試：<狀態>
   實作模型：<模型>　驗證模型：<待派>
   subagent：<有／沒有，沒有就寫原因>
   花多久／額度：<大約>
   卡過什麼：<或「無」>
   ```
6. 之後有進展（測試結果、修完），回一行加連結就好，不要整份重寫。回之前先看自己上一則說過什麼。

## 6. 要 Ray 決定時

不要只寫在聊天。照 `gh-handoff` skill：在 issue／PR 留言（開頭「## 結論」＋選項＋你的建議），貼 `needs-ray` label，然後回報派工的人「等 Ray：<連結>」。

## 7. 收尾清理

- PR 合併或關閉後，移除自己的 worktree。
- Xcode 專案：`git worktree remove` 之前，先刪 `~/Library/Developer/Xcode/DerivedData` 裡 `info.plist` 的 `WorkspacePath` 在那個 worktree 裡的資料夾，以及你用 `-derivedDataPath` 指定的資料夾。
- 只刪自己這張票的東西。

## 8. 磁碟

跑大工作（archive、大量下載、渲染）前先 `df -h /System/Volumes/Data`。剩不到 30 GB 就停下來回報，不要自己刪別人的檔案。

## 不做

不 merge、不推 main／release/*／tag、不關 issue、不改 milestone、不在沒被指名的 issue／PR 留言、不對外發送。
