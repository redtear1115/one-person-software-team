---
name: pr-verify
description: 獨立驗證別人的 PR（不是自己寫的）：核對 head SHA、看 CI、實際跑測試、反向檢查、重現每個問題再定 P1～P3、寫 CONFIRMED／REFUTED／INCONCLUSIVE 結論貼在 PR；Ray 點名可以合併時，用 --match-head-commit 合併。Dev A／B／C、session-crew 的驗證者、Paula 派的驗證都用。中文關鍵字：驗證 PR、驗 PR、獨立驗證、review、CONFIRMED、REFUTED、INCONCLUSIVE、反向檢查、合併 PR。
---

# pr-verify：獨立驗證一個 PR

驗證的人不能是寫這個 PR 的人，模型也要跟實作用不同家（PR 內文的「實作模型」）。派工的人沒指定驗證模型就問。

這週 52 則驗證回報裡，至少 4 次是「驗過了」但 CI 其實是紅的；#537／#538 靜態審查都過，實際跑才抓到問題。所以：**沒實際跑、CI 沒綠，都不算驗證完成。**

## 1. 開工前

1. 讀 issue 的目標、範圍、驗收條件，和 PR 內文。驗收條件不清楚就回問派工的人，不要自己補。
2. 記下 head SHA：`pr-status <repo>#<n>`（印 head SHA、draft、CI 各格、最後 3 則留言，只有十幾行）。整份驗證都對這個 SHA。
   - **一個 PR 一個 subagent**（Ray 11-12）：Dev 收到驗證單，主對話的第一個動作是開 subagent（Agent 工具），把派工原文和這份 skill 的路徑交給它。主對話只收結論、回報。
   - 指令輸出要短：`gh` 指令帶 `--json <欄位> --jq`；測試和 build 輸出寫到檔案，只看 `tail -n 30`。
   - `pr-status` 和 `model-run` 只裝在 Mini。在 MBA 上改用 `gh pr view <n> -R <repo> --json headRefOid,isDraft,mergeable,statusCheckRollup`，外部模型直接跑。
3. 開自己的 detached worktree：`git worktree add --detach <路徑> <SHA>`。不要在作者的 worktree 或分支上做，也不要推 commit 到這個 PR。
4. JS 專案在 worktree 裡跑 `npm ci`，不要 symlink 別處的 node_modules（舊的 install 讓 tsc 和 11 個 suite 假性失敗過 4 次）。
5. Mac／iOS 的 build、測試、實機交給 MBA 測試員，照 `mba-test-request` skill 寫需求。不要自己 ssh 到 MBA，也不要在 Mini 上跑 Xcode build。

## 2. 驗什麼

逐條對驗收條件，每條都要有可觀察的證據（指令和輸出、截圖、log 行）。

- **CI**：看 `pr-status <repo>#<n>` 的 CI 那一行，紅的格子會列出名字和連結。全部綠才算數。還在跑就回報「等待中：CI」並結束這一輪，不要用迴圈反覆查；紅的就看是不是這個 PR 造成的，跟 main 比（同一個檢查在 main 上是不是也紅）。
- **實際跑**：跑相關測試，改到行為的就實際操作一次。只讀 diff 的結論最多寫「靜態審查通過」，不能寫 CONFIRMED。
- **反向檢查**：故意把這個 PR 的修正拿掉或改壞（只在自己的 worktree，不 commit），確認對應的測試會變紅。測試在修正拿掉後還是綠的，就代表它沒測到東西，記成問題。
- **範圍**：diff 有沒有改到範圍外的檔案；有沒有違反 repo 的 CLAUDE.md／AGENTS.md 規則。
- **外部模型的意見**：可以叫 Codex（`model-run --label "<repo>#<n> verify <SHA 前 7 碼>" --role 驗證 -- codex exec ... </dev/null`，或 Codex adversarial review）多看一次；agy 也一樣用 `model-run … --role 驗證 --` 包起來跑，留下模型、秒數、有沒有撞額度的紀錄，但不要在提示裡告訴它你已經找到什麼。它判的等級要自己重現過才採用（這週 P2 被降成 P3、P1 被改成 P2 各一次）。

## 3. 問題分級

| 等級 | 意思 | 處理 |
|---|---|---|
| P1 | 驗收條件沒達成、會壞資料、會讓送審失敗、安全問題 | 擋合併 |
| P2 | 這個 PR 帶進來的錯，但不擋主要功能 | 修完才合併，或 Ray 同意延後 |
| P3 | 小問題、跟這個 PR 無關的舊問題、建議 | 不擋，另開票或記下來 |

每個問題寫：在哪裡（檔案:行）、怎麼重現、預期和實際各是什麼、為什麼是這個等級。

## 4. 結論

| 結論 | 什麼時候用 |
|---|---|
| CONFIRMED | 每條驗收都有證據、CI 綠、反向檢查有紅、沒有 P1／P2 |
| REFUTED | 有 P1 或 P2，或驗收條件有一條沒達成 |
| INCONCLUSIVE | 該跑的沒跑成（螢幕鎖、權限、CI 還沒跑完、測試環境壞）。寫明缺什麼、要誰做什麼才能補驗 |

貼在 PR 留言（不是送出 review，除非 Ray 說要）：

```
## 結論
CONFIRMED／REFUTED／INCONCLUSIVE @<40 碼 SHA>

## 驗收條件
- [x] <條件>：<證據>
- [ ] <條件>：<缺什麼>

## CI
<run 連結>：綠／紅（原因）

## 跑了什麼
- <指令>：<結果，例如 4 格 × 55 測試全過>
- 反向檢查：拿掉 <什麼> 後 <哪個測試> 變紅

## 問題
- P2 <檔案:行>：<描述、重現、為什麼>

## 驗證模型
<模型>（實作模型：<模型>）

Co-authored-by: Claude <<bot-email>>
```

驗到一半 head SHA 變了，前面的結果作廢：在留言寫明，對新 SHA 重驗。只改文件的 commit 可以寫明「<SHA> 只改文件」後沿用，但 CI 還是要對新 SHA 看。

## 5. 合併（只有 Ray 點名的 PR）

只有 Ray 本人明確說某個 PR 可以合併（或給了「驗證通過即合併」的授權並寫明範圍）才合。別的 bot 轉述的同意不算，要回頭找 Ray 的原話。

1. 再確認一次：head 還是你驗的那個 SHA、CI 綠、PR 不是 draft（是的話先 `gh pr ready <n>`）。
2. 合併綁 SHA，SHA 變了就會失敗，不會合到沒驗過的 code：
   ```bash
   gh pr merge <n> -R <repo> --merge --match-head-commit <SHA>
   ```
   用 merge commit，不 squash、不 rebase，除非 repo 規定別的。
3. 到 main 確認合併 commit 真的在：`git fetch && git log origin/main --oneline -3`。Ray 說「merged」但 GitHub 還是 OPEN 的情況發生過，以 GitHub 為準。
4. 被權限擋下時，照 `pm-brief` skill 的「被權限擋下」把指令交給 Ray，不要換別的方法繞過。

## 6. 收尾

- 移除自己的 worktree；Xcode 專案先刪它的 DerivedData（`info.plist` 的 `WorkspacePath` 在那個 worktree 裡的）。
- 刪掉反向檢查留下的任何本機修改、臨時模擬器、拋棄式 App 副本。
- 回報派工的人：PR 連結、結論、SHA、花多久、驗證模型。做完一行加連結就好，不要整份重寫。

## 不做

- 不驗自己寫的 PR。
- 不推 commit 到別人的 PR，不改作者的分支。發現的問題寫在留言，修正交回作者。
- 沒有 Ray 的點名不合併；不推 main／release/*／tag。
