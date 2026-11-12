# 範例：一則派工和它的回報

2026-11-12，專案經理 bot（Paula）派給開發 bot（Dev C）的真實訊息，以及 Dev C 的回報。原文照貼，只把機器名稱和信箱換成佔位符。repo 是私人的，連結外人打不開。

背景：一張 PR 的 CI 設定是舊版，只能在一台正在跑測試的 Mac 上執行，所以卡住了。Paula 請 Dev C 把 main 合進分支。

## 派工（Paula → Dev C）

【Paula 派工：mars-dawn#574（merge main 讓 CI 能在 Mini 跑；凍結路徑，先於 website#368）】
- 票：https://github.com/redtear1115/mars-dawn/pull/574 ；head ab58b6f，base 已是 main（#573 已合併），draft，MERGEABLE
- 問題：#574 分支上的 `.github/workflows/ci.yml` 還是舊版 `runs-on: [self-hosted, marsdawn, mba]`，CI run 43133029279 只肯上 MBA runner，而 MBA runner 暫停中（Quella 在 MBA 測這張），所以排了一小時不會動。main 的 ci.yml（#692）是 `[self-hosted, marsdawn]`，Mini 能接。
- 目標與驗收：
  1. 在 #574 的分支（`feat/506-openable-filter-devc`，你的分支）merge origin/main（merge commit，不 rebase、不 force push）。Dev A 已試過拋棄式 merge：無衝突。
  2. 確認 merge 後 ci.yml 的 `runs-on` 是 `[self-hosted, marsdawn]`，且對 origin/main 的 diff 只剩 #574 自己的檔案（列出檔名）。
  3. 本機 build-for-testing 過即可；offscreen 不用重跑整套（Dev A 在 ab58b6f 跑過四格各 196 全過，也在 merge 後跑過 signed 196×2）。要跑就拿 Mini 鎖、跑完立刻放。
  4. 看 Mini `SCREEN.md` holder，空就一般 push。新 push 會因 concurrency 取消卡住的舊 run，新 run 應該由 `<mini-runner>` 接；推完查一次，確認 job 的 labels 沒有 `mba`、有被 runner 接走。
  5. PR 內文「進度」段補：merge main 的原因、新 head SHA。
- 範圍／不做：只 merge，不改 code、不改 workflow 檔本身（用 main 的版本）。不合併、不標 Ready（CI 綠後我來標）。不要恢復 MBA runner。
- 實作模型：不需要模型判斷（無衝突 merge）；若出現衝突就停下回報，不要自己解。
- 現況：Dev A code CONFIRMED @ab58b6f（留言 6811450333）；Quella 的 MBA 實測在 ab58b6f 上進行中，不受影響。
- 執行方式（必做）：第一個動作就是開一個 subagent（Agent 工具），把這則派工原文和 dev-ticket skill 交給它做。主對話只派 subagent、收結論、回報我。
- 等待：不用 sleep／until 迴圈。推完確認 runner 接走就回報「等待中：CI」並結束。
- 回報：6 行內：新 head SHA、CI run 連結與是否被 Mini 接走、diff 檔名、卡點。

## 回報（Dev C → Paula，約 5 分鐘後）

#574 已 merge main，新 head 是 `465d4bb2cccc948ba60eacdfe9f0c1e9a0f90b2e`（原 `ab58b6f`，merge 前 main 是 `de6ca42`）。CI 還在排隊，Mini 的 runner 還沒接走這個 run。
- **CI：** run 43176705108，跑中（queued）。job `ci` 的 labels 是 `[self-hosted, marsdawn]`，沒有 `mba`。`<mini-runner>` 在線但正忙著跑 Dev A 的 `fix/681-background-handoff-deva`，所以先排隊。舊 run 43133029279 已 cancelled。
- **diff（對 origin/main）：** `DESIGN.md`、`MarsDawn/Resources/Localizable.xcstrings`、`MarsDawn/Sidebar/FileTreeViewController.swift`、`MarsDawn/Sidebar/SidebarFolderStore.swift`、`MarsDawnTests/SidebarOpenableFilterTests.swift`（新增）。ci.yml 不在其中，和 PR 原本的 5 個檔案一致。
- **本機測試：** `tools/test.sh` 沒有只 build 的模式，改跑最窄一格（`SidebarOpenableFilterTests`，未簽署、paywall 關、offscreen）。log 重讀過：9 個測試全過，`** TEST SUCCEEDED **`。Mini 鎖已放，holder 空。
- **做法：** 無衝突、一般 push、沒有 force，沒改 code 和 workflow。PR 內文原本沒有「進度」段，我新增了 `### 進度`（merge 原因加新 head），其他內文沒動。
- **卡點：** 無。

等待中：CI

Co-authored-by: Claude <<bot-email>>
