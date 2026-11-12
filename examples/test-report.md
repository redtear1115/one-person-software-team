# 範例：測試 bot 貼在 PR 上的結果

2026-11-12，測試 bot（Quella，在 MacBook Air 上）對 mars-dawn#684 的實機測試結果。原文照貼。

值得看的地方是「反向檢查」：把修正拿掉再跑一次測試，確認測試真的會變紅。這次抓到 2 個測試在舊 code 上也是綠的，等於沒測到東西。code review 看不出這種問題。

---

## 結論

#684 @ `d13c2a98761913da5484a9f7b85354679a845d86`（quella-645share，MBA）。**正向測試、全跑回歸和截圖都 PASS；反向檢查只成立一部分：有 2 個測試在舊 code 上不會紅，請 Dev A／B 看**：
- test02（`aWrongPasswordLeavesItOff`）在舊 code 上，iPhone 和 iPad 都是綠的。
- iPad 的 test04（`theSharePopoverStaysOnItsButtonAfterRotation`）在舊 code 上也是綠的。
- test01 照預期紅；test03 也紅了，但這不是單上寫的預期，原因見反向檢查。

- 時間：11-12 16:12 開工，17:05 完成（CST）。本 PR 已合併，測的是 Paula 指定的 head `d13c2a9`，也就是 Dev B CONFIRMED 的版本。
- 照 Paula 11-12 的修改：
  - 反向檢查只反轉 #684 自己的兩個 source 檔：merge base `4ac795e` 版的 `ShareToolbarButton.swift`、`DocumentViewerView.swift`，測試檔保留新的。
  - base 一律當成 main，所以回歸的基準是 merge base `4ac795e`（main 上 #684 之前的那一版）。
  - test04 另外在 iPad 上跑。
- 環境：macOS 27.2，Xcode 27.2（27B52），iOS 27.2 模擬器。每次都是 `tools/test-ios.sh` 自己建、自己刪的模擬器。
  - iPhone：預設的 iPhone-17。
  - iPad：`iPad-Pro-13-inch-M5-12GB`。

### 1. 正向：ShareButtonUITests：PASS
| 裝置 | test01 | test02 | test03 | test04 |
|---|---|---|---|---|
| iPhone 17 | 過 | 過 | 過 | 略過（不是 iPad） |
| iPad Pro 13（M5） | 過 | 過 | 過 | **過**（沒有略過） |

### 3. 反向檢查：部分成立
在 `d13c2a9` 開拋棄式 worktree，執行 `git checkout 4ac795e -- MarsDawnMobile/ShareToolbarButton.swift MarsDawnMobile/DocumentViewerView.swift`（只動這兩個檔，2 files changed, 3 insertions, 53 deletions），然後重跑 1。沒有推，用完就刪了。

| 測試 | iPhone（舊 code） | iPad（舊 code） | 預期 | 結果 |
|---|---|---|---|---|
| test01 鎖著不能分享 | **紅**：`XCTAssertFalse failed - off until it is unlocked` | **紅**（同左） | 紅 | 成立 |
| test02 密碼錯仍然不能分享 | 綠 | 綠 | 紅 | **不成立** |
| test03 解鎖後可以分享（對照） | 紅：`XCTAssertFalse failed - control: off while locked` | 紅（同左） | 綠 | 見下 |
| test04 轉向後 popover 還在按鈕下 | 略過 | 綠 | 紅 | **不成立** |

- test01 在舊 code 會紅，所以原因確實是按鈕狀態，不是「重現不了」。
- test03：`d13c2a9` 的 test03 在解鎖前多了一行 `XCTAssertFalse(share.isEnabled, "control: off while locked")`。舊 code 鎖著時按鈕是 enabled，所以這一行紅了。單上「test03 綠」的預期，是照 `fb728bd` 版的測試寫的。
- test02 在舊 code 是綠的：輸入錯的密碼之後，`isEnabled == false`，點了也沒有出現分享表，所以這個測試抓不到舊 code 的問題。原因我沒有查，請 Dev 看。
- iPad 的 test04 在舊 code 是綠的：iOS 27.2 模擬器上，舊 code 轉向之後，popover 也還在按鈕下，所以這個測試在這台抓不到 #612 的問題。下面第 5 項的截圖，是用新 code 實機轉向拍的。

### 4. 回歸：全跑，沒有新紅
| | 執行 | 紅 | 時間 |
|---|---:|---:|---:|
| 基準 `4ac795e`（main，#684 之前） | 35 | 3 | 833 秒 |
| PR `d13c2a9` | 39 | 3 | 838 秒 |

- xcdiff 比對：新紅 0、舊紅 3、只在 PR 4（就是 ShareButtonUITests 的 4 個，3 過 1 略過）。
- 3 個舊紅兩邊一樣，都不是這張 PR 造成的：
  - `LicenceGateUITests/test14_S3_regularWidthOutlineHasNoHeadingsWhileLockedThenShowsThemUnlocked`：`the side pane exists at regular width`
  - `SplitViewSpikeUITests/testSplitViewKeepsThePage`：`spike web=0 page=0 views=0 y=-`
  - `SplitViewSpikeUITests/testStackKeepsThePage`：同上
- 執行數 39 比基準多 4，沒有少。

### 5. 截圖：PASS
Debug build，語系用啟動參數 `-AppleLanguages` 設。
| 項目 | 結果 | 檔名 |
|---|---|---|
| de、iPhone 17，外觀與主題面板，半高 | 「Erscheinungsbild und Thema」完整，沒有被截成「…und…」，Fertig 在右邊 | `de-iphone17-theme-half.png` |
| de、iPhone 17，全高 | 同上 | `de-iphone17-theme-full.png` |
| de、iPad，popover | 標題完整，Fertig 在右邊 | `de-ipad-theme-popover.png` |
| en、zh-Hant、ja、ko、fr（另外補了 zh-Hans、es），iPhone | 標題都清楚可讀，fr 的標題比較長，改成靠左，但沒有縮小到難讀 | `<語系>-iphone17-theme.png` |
| 舊 code 的 de 對照 | 上次的 b10d-de | `old-b10d-de-half.png`、`old-b10d-de-full.png` |
| iPad 開分享，直轉橫，再橫轉直 | 3 張的箭頭都指著分享鈕 | `ipad-share-portrait.png`、`ipad-share-landscape.png`、`ipad-share-back-portrait.png` |
| 密碼 PDF 未解鎖 | 分享鈕是灰的 | `en-iphone17-locked-pdf-share.png` |

**觀察（不在這張的範圍，不判失敗）**：iPad 直向時，「外觀與主題」的 popover 出現在畫面上方中間（約 x 430～808 pt），沒有箭頭指著右上角的調色盤鈕（x 996）。見 `de-ipad-theme-popover.png` 和 `de-ipad-before-theme.png`。

### 證據與收尾
- iCloud `bot 交件/mars-dawn-684/d13c2a9/`：上面的截圖，以及 `tests/`（6 次的 JSON、ShareButtonUITests 的 log、`diff-full.md`）。
- 已清掉：模擬器（test-ios.sh 開的已經自己刪了，截圖用的 2 台我刪了）、我開的 DeviceHub、DerivedData、result bundle、3 個 worktree、素材。
- caffeinate 開著（pid 98738），Paula 說後面還有單，先不關。

Co-authored-by: Claude <<bot-email>>


