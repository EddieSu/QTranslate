# ADR-0002: 資料目錄預設放在程式旁邊，作業系統的使用者資料夾只當退路

上游把 QTranslate 設計成可攜式程式：設定、外掛、外掛資料和翻譯歷史紀錄都放在程式旁邊，整個資料夾搬走或備份就帶走一切。`AppDataDirectory.resolve()` 依序判斷：有 `-DappData` 就用它；否則用 jpackage 啟動器（`QTranslate.exe`）所在的資料夾或 JAR 所在的資料夾，只要「可寫」就用；都不行才退到作業系統標準位置（Windows 是 `%APPDATA%\QTranslate`），讓程式裝在不能寫的地方時不必提權也能執行。事後看會意外的地方有兩個：刪掉程式資料夾就等於刪掉使用者的設定與歷史紀錄；而 Java 在 Windows 上判斷資料夾可不可寫時只看唯讀屬性、不看權限，裝在 `C:\Program Files` 會被誤判成可寫，一般使用者實際寫不進去，設定每次啟動都回到預設值。SHEICO 因此採每位使用者各自安裝到 `%LOCALAPPDATA%\Programs\QTranslate\`，佈署腳本升級時只保留 `datastore\`、`plugins_data\`、`logs\`，其餘清掉重鋪。

**否決的做法**：一律放作業系統的使用者資料夾（現在只當退路）。

**來源**：`app/src/main/kotlin/com/github/ahatem/qtranslate/app/AppDataDirectory.kt`（檔頭說明與 `resolve()`）；commit c27c64e（2026-08-13，加入 jpackage 啟動器資料夾這一步）；佈署指南第 2.1 節（Windows 上 `canWrite()` 的陷阱）、第 3 節方案 A、第 6 節「升級會清掉這些」

---
🤖 opus · effort: xhigh · 2026-11-04
