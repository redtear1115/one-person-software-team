---
name: mba-test-request
description: 要請 MBA 測試員 Quella（測試員-quella@<mba>）跑 Mac／iOS 的 build、測試、模擬器、實機確認、截圖或 archive 時，用這個固定格式寫需求；也是 MBA 開工預檢、SCREEN-mba.md 鎖、排隊與收尾清理的規則。開發 bot 寫給派工的人、Paula 轉給測試員、測試員開工前都用。中文關鍵字：派測試、MBA 測試、跑測試、實機確認、archive、截圖單、MBA 鎖、螢幕鎖、預檢。
---

# mba-test-request：派測試給 MBA 測試員

MBA 測試員是 MacBook Air（`<mba>`）上的「測試員 Quella」，一次只跑一單，忙就排隊。Mini 上不跑模擬器或會搶畫面的測試。例外（Ray 11-07）：mars-dawn 的 `--part offscreen` 測試和反向檢查可由 Paula 派到 Mini 跑，要先拿 `SCREEN.md` 的鎖；kit 的 SwiftPM 測試也可以。

MBA 的 `~/.claude/skills/` 有這份 skill 和 pr-verify、asc-release、pm-brief、gh-handoff、doc-clarity、vault-diary 的**複本**。正本在 Mini 的 `~/Projects/claude/skills`（GitHub `redtear1115/bot-skills`）。改了這些 skill，commit 後跑 repo 裡的 `./sync-mba.sh`，它會 rsync 過去並比對 md5。

這週（10-26～11-01）MBA 測試停擺 10 次以上，原因幾乎都是：螢幕鎖著或睡著、鎖被佔著沒放、權限被系統更新重設、別人的 build 正在跑。所以下面的預檢每單都要做。

## 流程

**GitHub 是正本，Codync 傳話只是門鈴。** 單子、結果、討論都寫在 PR。Codync 網路斷過（兩台重開後互相找不到），傳話會丟，PR 不會丟。

1. 開發 bot：推分支、開 draft PR，用下面的格式回報派工的人（多半是 Paula）。
2. Paula：
   1. 把單子**原文貼成 PR 留言**，開頭寫「【測試需求】」。
   2. 在 PR 貼 `needs-mba-test` label（mars-dawn、mars-dawn-kit、dq-island、<org>/futari、<org>/charspin 都已經建好）。
   3. 用 message_bot 傳一句「<repo>#<n> 有測試單」給 `測試員-quella@<mba>`。傳不到也沒關係，測試員每 15 分鐘會自己查一次 label。
   4. 替它在 SCREEN-mba.md 拿鎖（見「鎖」）。
   5. 單子寫「需要畫面」的，派單前先請 Ray 讓 MBA 保持解鎖、不要操作（`pm-brief` B 段），不要等測試員跑到一半才發現螢幕鎖了。
3. 測試員：
   1. 收到門鈴，或 routine 查到 `needs-mba-test`（`gh search prs --owner redtear1115 --owner <org> --label needs-mba-test --state open`），就讀 PR 裡最新那則「【測試需求】」。
   2. 做完預檢才開跑。
   3. 跑完把結果**貼在 PR 留言**，把 label 從 `needs-mba-test` 換成 `replied-mba-test`，再用 message_bot 回報 Paula（傳不到就算了，PR 上已經有）。
   4. 截圖放 iCloud Drive 的 `bot 交件/<repo>-<號碼>/`，PR 留言只寫路徑。
4. 失敗：Paula 轉回原本的開發 bot。修完推新 commit，用新的 SHA 再派一次：再貼一則新的「【測試需求】」，把 label 換回 `needs-mba-test`。

## 需求格式（每一欄都要填）

```
【測試需求】
repo：redtear1115/<repo>
PR：#<n>（分支 <branch>）
commit：<完整 40 碼 SHA>
要跑什麼：
- <指令，例如 tools/test.sh --all --part offscreen>
- <實機或模擬器確認：裝置、iOS／macOS 版本、方向、要看的畫面>
需要畫面嗎：<不需要（offscreen）／需要（screen、實機、截圖）>
通過標準：
- <例如：全部測試通過；iPad 橫向時側欄不會蓋住內文>
- <完整跑時寫「格數 × 測試數」，並先跑 main 當基準比對>
結果寫在：PR #<n> 留言
回報給：<派工的 bot 名稱>
```

模擬器寫 UDID，不要只寫名稱（同名的會有好幾台）。

### 截圖單另外加這些欄位

截圖重拍過 4 輪、截到過含密碼的備忘錄，所以截圖單要寫清楚：

```
版本與 build：<版本 (build)>，Release（不要 Debug）
語系／主題：<例如 en、zh-Hant；淺色／深色>
狀態列：9:41、滿格、滿電（模擬器用 xcrun simctl status_bar override）
起始畫面：<打開哪個檔案、捲到哪裡>
尺寸：<例如 iPhone 2622×1206>
不能入鏡：個資、密碼、其他 App 的視窗、Codync
交件：每拍一張就放進 iCloud Drive 的 bot 交件/<repo>-<PR 或 issue 號>/，不要等整批
```

**交件資料夾**：Mini 和 MBA 用同一個 iCloud，Ray 用手機就看得到。路徑是 `~/Library/Mobile Documents/com~apple~CloudDocs/bot 交件/<repo>-<號碼>/`，例如 `bot 交件/mars-dawn-556/`。同一張單重拍時，加後綴 `-r2`、`-r3`。不要直接放在 iCloud 根目錄。PR 合併或關閉後，Ditto 會列出可以刪的資料夾，請 Ray 確認後再刪；其他 bot 不要自己刪。

只在目標 App 在前景時截。需要輔助使用權限或模擬器 Apple ID 登入的，先請 Ray 備妥再開工。

### 測試員怎麼拍（截圖單）

能用指令做、能用指令檢查的都用指令，不要用眼睛看「差不多」。

1. **只截模擬器，不截整個螢幕**：用 `xcrun simctl io <UDID> screenshot <檔名>.png`。這樣只會拍到那台模擬器的畫面，不會拍到備忘錄、別的 App 或 Codync。不要用 `screencapture` 或快捷鍵截整個螢幕。Mac App 用 `screencapture -l <視窗 id>` 只截那個視窗。
2. **狀態列**：開拍前 `xcrun simctl status_bar <UDID> override --time "9:41" --dataNetwork wifi --wifiBars 3 --cellularBars 4 --batteryState charged --batteryLevel 100`，拍完 `xcrun simctl status_bar <UDID> clear`。
3. **語系和主題**：用指令設，不要手動點。主題用 `xcrun simctl ui <UDID> appearance light|dark`。語系用啟動參數 `xcrun simctl launch <UDID> <bundle id> -AppleLanguages "(zh-Hant)" -AppleLocale zh_TW`。
4. **Release build**：用 `-configuration Release` 建置。裝之前先查 `Info.plist` 的版本號跟單子寫的一樣：`/usr/libexec/PlistBuddy -c "Print CFBundleShortVersionString" -c "Print CFBundleVersion" <app>/Info.plist`。
5. **每拍一張就檢查尺寸**：`sips -g pixelWidth -g pixelHeight <檔名>.png`，跟單子寫的尺寸對不上就重拍，不要自己縮放。
6. **個資**：模擬器裡只放測試帳號和假資料，不登入 Ray 的 Apple ID 或個人帳號。截到通知或別人名字的，刪掉重拍。
7. 檔名寫 `<語系>-<裝置>-<序號>-<畫面>.png`，每拍一張就放進單子指定的資料夾，最後在 PR 留言列出檔名和尺寸。

## 開工預檢（測試員每單都做）

0. **負載**：`uptime` 看 load average。MBA 負載高時，SSH 和 Codync 都會像斷線（11-01 負載到 88，Spotlight 在索引 runner 目錄，排除後才降下來）。load 超過 20 就先 `top -l 1 -o cpu -n 5` 看是誰；是 `mds`／`mdworker`（Spotlight）就回報 Paula，請 Ray 把那個目錄加進 Spotlight 的隱私清單。不要自己殺系統 process，也不要在高負載下開跑，跑出來的時間和逾時都不準。
1. **沒有別人在跑**：`pgrep -fl 'xcodebuild|test.sh'` 沒有結果，`ls ~/marsdawn-remote/.lock` 不存在。有的話先回報「排隊中」，不要殺別人的 process。CI runner（mars-dawn、charspin）也會在 MBA 上跑 xcodebuild。
2. **螢幕醒著、沒鎖**：需要畫面的項目（`--part screen`、沒加 `--part`、實機、截圖），螢幕必須解鎖，而且 Ray 閒置夠久：`test.sh` 至少 30 秒，操作 App 至少 120 秒。閒置秒數用 `ioreg -c IOHIDSystem | awk '/HIDIdleTime/ {print int($NF/1e9); exit}'` 看。螢幕鎖著就停下來回報 Paula「等 Ray 解鎖 MBA」，不要硬跑，硬跑的結果只會是 INCONCLUSIVE。
3. **不讓螢幕睡**：整段跑在 `caffeinate -du` 底下，例如 `caffeinate -du tools/test.sh ...`。螢幕睡著時 test.sh 會拒跑；已經睡了就先 `caffeinate -u -t 5` 叫醒。
4. **權限還在**：系統更新會重設權限（10-30、11-01 各一次）。確認跑測試的終端機或 Codync 仍有「完整磁碟取用」（讀得到 `~/Library/Containers`）和「輔助使用」（實機操作、截圖）。沒有就停下來請 Ray 到系統設定開，不要空等權限視窗。
5. **磁碟**：`df -h /System/Volumes/Data`，剩不到 30 GB 先回報。
6. **鑰匙圈**：archive、簽名要用到鑰匙圈。透過 SSH 時讀不到鎖住的登入鑰匙圈，要 Ray 在 MBA 上解鎖，不要替 Ray 輸入密碼。

## 鎖：SCREEN-mba.md

- 鎖檔在 **Mini** 的 vault：`<vault>/MarsDawn/協作/SCREEN-mba.md`。規則跟 Mini 的 `SCREEN.md` 一樣，開工前讀一次。
- **MBA 讀不到這個檔**（MBA 上沒有 vault）。所以由 Paula 代 Quella 拿鎖和放鎖：派單時寫 `holder`，Quella 回報後清掉，並在 Log 加一行。
- 只有 `holder` 空著，而且 `queue` 空著或你排第一，才能拿鎖。決定要等的那一刻就排進 `queue`，一行一個 `- ` 項目。
- **排進 `queue` 之後就不要等**。回報派單的人「等待中：MBA 鎖，排第 N」，然後結束這一輪。輪到你時，Paula 會再傳訊息叫你。不要寫 `while`／`until`／`sleep` 迴圈去讀 `holder`，也不要反覆開這個檔案看。在幾十萬 token 的對話裡，每查一次都要把整段對話重讀一遍（Ray 11-09）。
- `holder` 寫自己的名字。放鎖前先確認 `holder` 還是你，再清掉，並在 Log 加一行，時間用 `$(date)`。
- **斷線時怎麼放鎖**：拿鎖的 session 或 bot 中斷了（10-30 曾空佔 8 小時），Paula 先確認 MBA 上沒有它的 process（`pgrep -fl 'xcodebuild|test.sh'`），再清掉 `holder`，並在 Log 寫「代為釋放：<原 holder>，原因」。還有 process 在跑就不要清。
- 不插隊。只有擋送審的 release gate 可以排到最前面，而且要在 Log 寫原因。
- 不需要鎖的：不開視窗的 kit `swift test`（在 /tmp clone 裡跑）。其他 `tools/test.sh` 和實機操作都要鎖。

## archive 和上傳

- 送審的 build 必須用 App Store Connect 收的正式版 Xcode。10-31 用 27.2 封存的 build 被 ASC 以 ITMS-90534 退件（CharSpin、Futari 各一次），當時改在 MBA 用 27.1 封存、回 Mini 簽名上傳。
- **注意（11-01）**：MBA 和 Mini 都只留一份 Xcode 27.2（27B52），App Store 自動更新已關，不另裝舊版（Ray 決定）。archive 前先跑 `xcodebuild -version`，上傳前用 `xcrun altool --validate-app` 驗；還是 90534 就停下來問 Ray，不要送，也不要自己裝別版 Xcode。
- archive 在 MBA 做，再回 Mini 簽名、匯出、上傳時，匯出後要核對 entitlements（Game Center、iCloud、Mac sandbox 曾經在匯出後漏掉）：`codesign -d --entitlements - <app>`。
- 上傳是對外動作，要派工的人或 Ray 同意。各專案的步驟看 `asc-release` skill。

## 收尾（每單都做）

- 刪掉這單建的 clone 或 worktree、它的 DerivedData（`-derivedDataPath` 指定的，或 `info.plist` 的 `WorkspacePath` 在那個資料夾裡的）、臨時模擬器（`xcrun simctl delete <UDID>`），還有拋棄式 bundle id 的 App 副本和它的 container。
- 不是這單建的不要刪。刪不掉的 container 空殼列給 Ray。
- 不要啟動、修改或關掉 `/Applications/MarsDawn`，那是 Ray 的 TestFlight 版。
- 結果貼 PR：`## 結論`（通過／失敗／INCONCLUSIVE）、測的 SHA、跑了哪些指令、格數 × 測試數、失敗的測試名、log 重點、清了什麼。結尾加 `Co-authored-by: Claude <<bot-email>>`。

## 常見錯誤

- 只寫「幫我測一下」：測試員不知道跑哪些。
- 沒給 SHA：分支在測試途中又被推了新 commit，結果對不上。
- 一半在 Mini 跑、一半在 MBA 跑：整段都交給測試員。
- 結果只回在聊天：開發 bot 和 Ray 看不到。一定要貼在 PR。
- 輸出只留最後 30 行：失敗原因常在前面。完整 log 存檔，PR 貼重點和檔案位置。
