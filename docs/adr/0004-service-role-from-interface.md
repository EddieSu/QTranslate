# ADR-0004: 服務角色由實作的介面決定，不由外掛宣告

服務屬於哪個角色（翻譯器、字典……）只看它實作了哪個介面：實作 `Translator` 就是翻譯器，同時實作 `Dictionary` 就兩個角色都有，外掛不必也不能另外宣告。這是兩輪修正的結果。最早是「依序判斷型別」，只會得到一個答案，既能翻譯又能查字典的服務只會被歸到先判斷到的那一類。外掛 API v2（2026-08-15）改成由服務自己宣告角色，但宣告和實作是同一件事的兩種說法：兩者不一致時，主程式照宣告挑出服務再轉型成介面，使用者第一次用就丟 `ClassCastException`，還得另外寫一個檢查器去攔。隔天（2026-08-16）改回從介面推斷，但改成逐一比對每個角色、不再取第一個符合的，所以一個服務擁有它實作的每一個角色，檢查器也一起拿掉。只影響「怎麼做」的可選能力（例如選聲音、批次翻譯）刻意不算角色，用額外的介面表示。角色列舉屬於公開的 `:api`，新增角色只需升外掛 API 的次版號。

**否決的做法**：依序判斷型別、只取一個角色；由服務宣告角色，再另外寫檢查器驗證。

**來源**：`api/src/main/kotlin/com/github/ahatem/qtranslate/api/plugin/ServiceRole.kt` 檔頭說明；`api/src/main/kotlin/com/github/ahatem/qtranslate/api/plugin/Service.kt` 檔頭說明；commit 4f7bc73「Plugin API v2 (#167)」（2026-08-15，改成宣告並加上 CapabilityValidator）；commit 13f688f「API v2: derive service roles from interfaces instead of declaring them (#184)」（2026-08-16）

---
🤖 opus · effort: xhigh · 2026-11-04
