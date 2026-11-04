# ADR-0003: 服務 id 由主程式組成「外掛:實例:服務」，實例欄先固定填 default

外掛 API v2（2026-08-15）以前，每個服務自己宣告全域唯一的 id，設定檔直接存這個字串。v2 改成服務只宣告在自己外掛內唯一的 key，由主程式組成 `外掛id:實例id:服務key`（例如 `google-services:default:google-translator`），服務預設、停用清單和翻譯歷史紀錄存的都是組好的字串。這樣同一個外掛才能用不同帳密裝兩份，兩份宣告同一個 key 也分得開。現在還不支援多實例，但實例欄從一開始就固定填 `default`，日後開放多實例時 id 的形狀不必再改。代價是所有舊設定裡的 id 一夕失效，要靠 `LegacyServiceIds` 這張凍結的對照表遷移；它只認得十個內建外掛，第三方外掛的舊選擇會遺失。手寫設定檔或種子檔時，也必須寫完整的三段式 id。

**否決的做法**：讓外掛自己宣告全域唯一的服務 id。

**來源**：`core/src/main/kotlin/com/github/ahatem/qtranslate/core/plugin/registry/ServiceId.kt` 檔頭說明；`core/src/main/kotlin/com/github/ahatem/qtranslate/core/settings/data/LegacyServiceIds.kt` 檔頭說明；commit 4f7bc73「Plugin API v2 (#167)」（2026-08-15）；佈署指南第 5 節「服務 id 的寫法」

---
🤖 opus · effort: xhigh · 2026-11-04
