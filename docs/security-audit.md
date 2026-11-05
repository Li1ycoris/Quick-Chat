# 安全稽核紀錄

- 日期：2026-11-05
- 範圍：commit `ee5c098`（純文字化之前）的完整原始碼、建置腳本、entitlements、GitHub Actions
- 結論：**沒有發現後門**。程式行為與宣稱的一致：一個 chatgpt.com 浮動視窗，加上管理原生 ChatGPT app 的快捷鍵。

## 1. ChatGPT account 操作
- Web view 只載入 `https://chatgpt.com/`。Session 存在 `WKWebsiteDataStore.default()`，跟一般瀏覽器一樣。
- 登出功能會用 `removeData(ofTypes: allWebsiteDataTypes())` 清除所有網站資料。
- Native 程式碼不會讀取、儲存或傳送帳密、cookie、token。
- 唯一的 JS 注入是 `document.querySelector('#prompt-textarea')?.focus()`。
- User-Agent 偽裝成 Safari，目的是讓網站正常顯示，也讓 Google 登入可以用，沒有危害。

## 2. OAuth
- App 沒有自己實作 OAuth，沒有 client secret，也不處理 redirect URI。
- `inPanelHosts` 白名單讓 SSO 流程留在面板內，其他連結會用外部瀏覽器開啟。
- `staysInPanel` 的 suffix 比對是正確的：`notgoogle.com`、`google.com.attacker.com` 都不會誤判成符合。
- 建議：白名單範圍偏寬（整個 `google.com` 等），可以縮小到登入用的子網域（見 `TODO.md`）。

## 3. 安全性與後門
沒有找到：
- `URLSession`、socket，或任何 app 自己發出的網路連線
- `Process`／`NSTask`／`system()`／`osascript`／shell 執行
- 讀取 keychain、cookie、密碼的程式碼
- telemetry、analytics、遠端設定、自動更新
- `dlopen`、`eval`、base64 解碼後執行等動態載入

做得好的地方：
- GitHub Actions 用 commit SHA 固定版本，會驗證版本號格式，權限只開 `contents: write`，並驗證簽章、產生 sha256。
- 收到 SIGTERM／SIGINT／SIGHUP 時會先還原 keymap 再結束。
- 改寫 `~/.codex/keybindings.json` 時只動自己負責的 command，會先備份、加上 marker，停用或結束時還原。

需要知道的地方（不是後門）：
- **沒有開 App Sandbox**：因為要改寫其他 app 的設定檔，技術上必須這樣，但代表 app 有一般使用者等級的檔案存取權。
- **Ad-hoc 簽章**：沒有 notarization，下載的檔案只能靠公布的 sha256 驗證完整性。

## 後續變更
- `2489d23` 純文字化：移除上傳、下載、鏡頭/麥克風，以及 camera/audio-input entitlements 和 Info.plist
  的 usage description，進一步縮小權限範圍。現在 app 不需要任何系統權限。
