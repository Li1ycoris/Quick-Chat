# TODO

狀態：`[ ]` 待辦　`[~]` 等待中／需確認　`[x]` 完成

## 進行中

### [~] 預設使用 Instant 模型
- **需求**：Quick Chat 開啟時就使用 ChatGPT 的 Instant 模型（追求速度）。
- **卡在**：要等使用者在本地安裝、測試後，提供正確的 model slug。
- **作法**：
  1. 使用者在 ChatGPT 模型選單切到 Instant，從網址列複製 `?model=` 後面的值
     （也可以試 `https://chatgpt.com/?model=<slug>`，看選單是否顯示 Instant）。
  2. 在 `QuickChatPanel.swift` 新增一個命名清楚的常數（例如 `instantModelSlug`），
     讓 `chatURL` 帶上 `?model=<slug>`。`chatURL` 在 `signOut()`、`makePanel()`、
     `webViewWebContentProcessDidTerminate` 都會用到，所以只改常數這一處就好。
  3. 加註解：slug 沒有官方文件，OpenAI 改模型名稱時（例如 GPT-5.3 → 5.5 Instant）要跟著更新。
- **注意**：
  - 塞錯 slug 不會報錯，會默默退回預設模型，所以一定要實測確認。
  - 社群回報過 `?q=` 搭配 `?model=` 時，model 參數會被忽略。
  - 截至 2026-05，GPT-5.5 Instant 已經是 ChatGPT 的全站預設模型，所以釘選主要是防止帳號記住其他模型（例如 Thinking）。

### [~] 本地驗證純文字化（commit `2489d23`）
在 macOS 上 `make run` 後確認：
- [ ] 能正常編譯（cloud 端沒辦法 build，這是第一次真正編譯）
- [ ] 登入、聊天正常
- [ ] ⌘U／附件按鈕不會跳出檔案選擇器
- [ ] 點下載或生成的檔案不會存檔，也不會把聊天畫面換掉
- [ ] 不會要求麥克風或相機權限
- [ ] Google／Apple／Microsoft 登入流程仍然留在面板內

## 建議（尚未決定要不要做）
- [ ] **縮小 OAuth 白名單**：`inPanelHosts` 現在包含整個 `google.com`、`apple.com`、`microsoft.com`、`live.com`。
      可以改成只列登入用的子網域（例如 `accounts.google.com`、`appleid.apple.com`、`login.microsoftonline.com`、
      `login.live.com`）。改完要實測三種 SSO，確認重新導向的網域都有涵蓋。
- [ ] **發佈改用 Developer ID 簽章並送 notarization**：現在是 ad-hoc 簽章，使用者要手動略過 Gatekeeper。

## 完成
- [x] 安全稽核：ChatGPT account 操作、OAuth、後門 → 見 `docs/security-audit.md`
- [x] 純文字化：移除上傳、下載、鏡頭/麥克風、相關 entitlements／Info.plist／CI 檢查，並更新文件（`2489d23`）
