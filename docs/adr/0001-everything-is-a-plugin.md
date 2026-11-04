# ADR-0001: 所有服務都做成外掛，主程式不內建任何服務

原版 QTranslate（Questsoft）停止開發後，它依賴的翻譯 API 一改就整個壞掉、沒人能修。上游作者 ahatem 用 Kotlin 從頭重寫時，唯一刻意改變的核心設計就是「一切皆外掛」：翻譯引擎、文字辨識、語音合成、拼字檢查、字典全都是執行時才安裝的獨立 JAR，主程式只透過 `:api` 模組的介面跟它們互動，外掛也只准依賴 `:api`。哪個服務改了 API 或關站，換掉那一個外掛就好，主程式照常運作。代價是 `:api` 一公開就是承諾，不相容的修改要升外掛 API 的主版號、所有外掛都要重新編譯；不含外掛的純程式 JAR 連一句都翻不了。SHEICO 的內部包也是靠這個設計，直接從套件裡拿掉會把文字送到外部的外掛 JAR，來落實公司的資料不外流政策。

**否決的做法**：像原版那樣把翻譯服務寫死在主程式裡。

**來源**：`README.md` 開頭介紹（原版停擺的經過與「everything is a plugin」那段）；`wiki/Architecture.md`「:api — the plugin contract」；`api/src/main/kotlin/com/github/ahatem/qtranslate/api/core/ApiVersion.kt`（版本相容規則）；佈署指南第 5 節「服務選擇」（SHEICO 內部包移除外掛的做法）

---
🤖 opus · effort: xhigh · 2026-11-04
