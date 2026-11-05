# Quick Chat — Claude 工作指引

macOS menu-bar app（Swift 6 / SwiftPM，macOS 14+）：全域快捷鍵叫出浮動面板，裡面是 `chatgpt.com` 的 `WKWebView`。
另外可解除原生 ChatGPT app（bundle id `com.openai.codex`）的 Show Mini 快捷鍵（改寫 `~/.codex/keybindings.json`，可還原）。

## 工作流程
- 需求在 cloud session（claude.ai/code）提出，在本地 Claude CLI 執行與測試。兩邊靠這個 repo 同步脈絡：
  - 待辦事項 → `TODO.md`（做完就更新狀態，不要只留在對話裡）
  - 安全稽核結果 → `docs/security-audit.md`
- 開發分支：`claude/admiring-goodall-qgq13o`
- 只有 macOS 能 build（cloud container 沒有 Swift toolchain）。本地驗證用 `make run`。

## 產品決策（已定案，不要回頭改）
- **Quick Chat 只做純文字。** 上傳檔案、下載檔案、鏡頭/麥克風（語音、拍照、視訊）全部停用。
  不要重新加回 `WKDownloadDelegate`、`NSOpenPanel`、media capture 權限，也不要加回 camera/audio-input
  entitlements 或 Info.plist 的 usage description。要用附件的人請他們改用瀏覽器或桌面 app。
- App 不需要任何系統權限（不需要 Accessibility、Automation、麥克風、相機），之後也維持這樣。

## 安全原則（來自稽核）
- Native 程式碼不能讀取、儲存或傳送帳密、cookie、token。登入完全交給 web view。
- 唯一允許的 JS 注入是 `focusComposer()`（focus 輸入框）。不要加入會讀取頁面內容的腳本。
- 除了 web view 載入 `chatgpt.com`，不要加任何對外網路連線（不加 telemetry、analytics、更新檢查）。
- Host 比對一律用 `host == d || host.hasSuffix("." + d)`，不要用 substring 比對。

## 主要檔案
- `Sources/QuickChat/Features/QuickChat/QuickChatPanel.swift` — web view、導航/下載/權限政策、面板
- `Sources/QuickChat/Core/ChatGPTApp.swift` — 啟動/重啟原生 ChatGPT app
- `Sources/QuickChat/Core/Keymap/` — 安全改寫 `keybindings.json`（備份 + marker + 還原）
- `scripts/build-app.sh`、`scripts/QuickChat.entitlements`、`.github/workflows/release.yml`
