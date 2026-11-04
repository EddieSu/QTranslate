# ADR-0005: SHEICO 內部版從 fork 建置，上游正式版收錄輸入法修正後就退回官方包

2026-08-29 現場回報：用注音在主視窗輸入框打字，還沒選字的注音會永久留在框裡（打「測試」變成「ㄘㄜ測ㄕ測試」），原因是上游把狀態寫回輸入框時蓋掉了輸入法組字中的內容（上游 issue 233）。我們送的修正（PR 234）被上游作者改寫成 PR 333，2026-10-03 才合進 develop，09-29 發行的 v1.5.0 沒有收錄；偏偏 OP 1511 的翻譯失敗（Google 主要網址被限流後，1.4.1 讀不懂備用網址的回應）只有 1.5.0 修好，沒有任何官方版本同時有兩個修正。因此決定內部版一律從 fork（EddieSu/QTranslate）建置：分支 `sheico/<上游版號>-ime-fix` 等於上游正式版 tag，加上 cherry-pick 輸入法修正，再把 `APP_VERSION` 改成 `<上游版號>-sheico.N`，沒有其他改動；推同名 tag 讓 fork 的 Release workflow 建出 Windows 套件。走 CI 是因為本機沒有 JDK，官方套件的執行環境也拆掉了 `java.exe`，本機建不了。1.5.0 起 Release workflow 要求 tag 和 `APP_VERSION` 完全一致，所以後綴必須寫進常數；Release workflow 不跑單元測試，靠比對修正後的檔案與上游一致，來沿用上游 CI 對 PR 333 的測試結果。`1.5.0-sheico.1` 在版本比較上會被當成比官方 `1.5.0` 舊，所以種子檔關掉自動檢查更新，也要告訴使用者不要自己升級到官方版。上游發行收錄 PR 333 的版本後就改回官方包，fork 只留著備查；確認方法是該版 tag 的 `AdvancedTextPane.kt` 裡找得到 `isComposingText`。

**否決的做法**：直接拿官方 v1.5.0 包重打包（沒有輸入法修正，中文打字會壞回去）；留在 1.4.1-sheico.1（修不掉 OP 1511 的翻譯失敗）。

**來源**：worklog 2026-W35「2026-08-29 · QTranslate · 中文輸入法組字被吃掉：定位到上游 race，patch + 回歸測試進 fork」與「2026-08-29 · QTranslate · 上游 PR 送出，內部包 1.4.1-sheico.1 上架共用資料夾」；worklog 2026-W44「2026-10-26 · QTranslate · OP 1511 翻譯失敗：上游 Google 外掛備用網址讀取缺陷，決定升級 1.5.0＋輸入法修正」；佈署指南第 7 節與第 8 節「更新提示」（`D:\tahua2025\eddiesu.tw\CODING\SHEICO\QTranslate\QTranslate-大量佈署指南.md`，不在這個 repo）；分支 `sheico/1.5.0-ime-fix`（commit 1db753d、8deea38）與 tag `v1.5.0-sheico.1`

---
🤖 opus · effort: xhigh · 2026-11-04
