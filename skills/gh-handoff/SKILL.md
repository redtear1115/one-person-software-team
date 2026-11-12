---
name: gh-handoff
description: Ray 的 bot 團隊在 GitHub 上交接工作的共同規則：只在被點名的 issue／PR 留言、留言格式、needs-*／replied-* label 怎麼換、要 Ray 決定時用 needs-ray、轉述 Ray 的話、給 Aria／Daphne 開票的範本、多語文案的核准與套用流程、inbox 分支、署名。任何 bot 要在 redtear1115/* 的 issue 或 PR 留言、貼 label、開 needs-copy／needs-ux 票、套用文案、交設計或文案檔時使用。中文關鍵字：GitHub 交接、留言、label、needs-ray、等 Ray 決定、交檔、文案核准、多語、套用文案、逐字比對。
---

# gh-handoff：在 GitHub 上交接

GitHub 是工作的正本。決定、測試結果、驗證意見都寫在 issue／PR，聊天訊息只用來叫人，內容附連結。

## 什麼時候可以留言

- 只在你被點名的 issue／PR 留言（被派工、被貼對應的 `needs-*`、Ray 指定）。
- 不主動掃全庫，不在別人的 PR 推 commit。

## 留言格式

```
## 結論
<一兩句：做了什麼／結果／建議>

<細節：可勾選的建議、風險、待確認分開寫>

Co-authored-by: Claude <<bot-email>>
```

不要加 Anthropic 預設的 noreply 署名。

## Label 狀態機

| 誰 | 收到 | 做完換成 |
|---|---|---|
| Daphne（設計） | `needs-ux` | `replied-ux` |
| Una（情境） | `needs-ux`（情境部分） | `replied-ux` |
| Aria（文案／多語） | `needs-copy`、`needs-i18n` | `replied-copy`、`replied-i18n` |
| 任何 bot | 要 Ray 決定 | 貼 `needs-ray` |
| Ray | 回覆完 | 拿掉 `needs-ray` |

- 只改你這次完成的那一類。沒改 label 視同沒做完。
- `idea`：Una 每週一開，Paula 每週二排序，Ray 拍板。

## 要 Ray 決定：needs-ray

1. 在該 issue／PR 留言，照上面的格式，但「## 結論」後面列出：
   - 要決定什麼（一句話）
   - 選項 A／B／C，各自的影響
   - 你的建議和理由
   - Ray 沒回時的預設做法
2. 貼 `needs-ray` label（redtear1115 的主要 repo 都已經建好）。
3. 聊天裡只回報派工的人「等 Ray：<連結>」。不要叫別的 bot 替你轉問。留言最後寫明怎麼回，例如「同意請回『#123 ok』，或在這裡留言」。Ray 在聊天裡按的綠勾或其他表情，bot 看不到，不算回覆。
4. Ray 在 GitHub 回覆並拿掉 label 後再繼續。
5. **Ray 在聊天裡決定的**：在 issue 留一則「## 決定」，寫 Ray 的原話、日期、決定了什麼、範圍，再拿掉 `needs-ray`，並在同一則寫明「label 由 <你的名字> 代移」。

## 轉述 Ray 的話

- 附原話和時間，再寫你的理解，兩者分開。例如：「Ray 10-26 原話：『實機測試 ok』。我的理解：iOS 1.9.0 內測 build 的實機測試通過，不含 #1822。」
- 只轉述原話涵蓋的範圍，不要擴大（「實機測試 ok」曾被轉成「內測和 #1822 都通過」；#1897 的轉述跟 Ray 說的相反）。
- 別的 bot 轉述的同意不算 Ray 的同意。要動手的事（合併、送審、刪除）回去找 Ray 的原話。

## 給 Aria／Daphne／Una 開票

```
## 要什麼
<一句話>

## 位置
- <檔案或頁面、畫面>：<現在的字或樣子>
（附截圖：放 design/refs/<issue號>/）

## 限制
<字數、語系、不能改的詞、尺寸>

## 怎麼回
在這則留言回 <表格：位置｜現在｜建議>，改完換成 replied-copy／replied-ux

相關：<PR／issue 連結>
急不急：<不急／某日前>　要重建置嗎：<要／不要>
```

一個 issue 一個請求。label 貼 `needs-copy`、`needs-i18n` 或 `needs-ux`。

## 多語文案：核准 → 翻譯 → 套用 → 比對

1. **en 和 zh-Hant 先給 Ray 核准**。Aria 在 issue 留兩語對照表，貼 `needs-ray`。Ray 的核准原話記在 issue 和之後的 PR 內文。
2. 核准後 Aria 才翻其他語系（zh-Hans、ja、ko、de、fr、es），照核准版翻。
3. **套用時不要手抄**：用程式從那則留言讀進表格（`gh api repos/<repo>/issues/comments/<id>`），在原始碼找「現在」欄**唯一**相符的字串再替換。找到 0 處或超過 1 處就停下來回報。
4. 字數用程式算（Python `len()`）。ASC 文字每語系最多 4000 字元；fr 在 `:` `?` `!` 前用不換行空格 U+00A0。
5. **套用後逐字比對**：用程式把每個語系、每個位置的最終字串跟核准表比對，結果貼在 PR。驗證的人再比一次。
6. 同一句話出現在好幾個地方（App、官網、商店頁、llms 文件），開票時就列出全部位置，一次改完（website#333、#345 漏改過）。

查目前等 Ray 的清單：

```bash
gh search issues --owner redtear1115 --label needs-ray --state open
gh search prs --owner redtear1115 --label needs-ray --state open
```

## 交檔

- 設計／文案檔：開分支，放在 `design/inbox/<issue號>-<短名>.*`，開 **draft PR**，不要自己 merge。
- 參考檔優先看 `design/refs/<issue號>/`。

## 不做

不 merge、不推 main／release/*／tag、不關 Ray 開的 issue、不改 milestone、不對外發送。
