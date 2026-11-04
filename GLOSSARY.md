# QTranslate（SHEICO fork）

桌面翻譯工具：在任何程式裡選取文字、按快捷鍵就跳出譯文，另外有文字辨識、語音合成、拼字檢查、字典、摘要、改寫與文件翻譯。上游是 ahatem/QTranslate（用 Kotlin 重寫的開源版，MIT 授權），所有服務都做成可以單獨安裝的外掛；這個 fork（EddieSu/QTranslate）只用來建置 SHEICO 的內部版，在上游正式版上補中文輸入法修正。

術語寫法是「中文術語（程式裡的英文識別字）」。中文術語盡量跟程式內建的繁體中文介面（`languages/zh-TW.toml`）一致；英文識別字只為了對照程式（類別、列舉值、設定欄位），文件和溝通一律用中文術語；程式裡沒有對應識別字的術語不加括號。每條底下的 _Avoid_ 列的是同一件事在程式、文件或介面裡出現過的其他說法：意思相同，但寫文件時不要用；有例外就寫在括號裡。

## 外掛與服務

**外掛**（`Plugin`）：
執行時才安裝的一個 JAR 檔，提供一個或多個服務。翻譯引擎、文字辨識、語音合成、拼字檢查、字典全都是外掛，主程式本身不帶任何服務（見 ADR-0001）。

**外掛描述檔**（`plugin.json`、`PluginManifest`）：
每個外掛 JAR 裡必備的描述檔，記外掛 id、名稱、版本、作者和最低 API 版本。外掛 id 是設定與外掛資料的永久鍵，不能改；它不一定等於 JAR 檔名，例如 AI Services 的外掛 id 是 `ai-plugin`，JAR 卻叫 `ai-services-plugin.jar`。

**外掛 API 版本**（`ApiVersion`、`minApiVersion`）：
外掛與主程式之間那份介面契約的版號。外掛宣告它需要的最低版本；主版號必須和主程式相同、次版號不能比主程式新，否則拒絕載入。

**服務**（`Service`）：
外掛提供、讓使用者挑選的一項功能，例如 Google 翻譯或 Google 語音合成。一個外掛通常提供好幾個服務。

**服務角色**（`ServiceRole`）：
服務的用途，共八種：翻譯器（`TRANSLATOR`）、語音合成（`TTS`）、文字辨識（`OCR`）、拼字檢查器（`SPELL_CHECKER`）、字典（`DICTIONARY`）、摘要產生器（`SUMMARIZER`）、改寫器（`REWRITER`）、圖片搜尋（`IMAGE_SEARCH`）。服務實作了哪個角色的介面就擁有那個角色，一個服務可以同時擁有好幾個（見 ADR-0004）。

**服務 id**（`ServiceId`）：
主程式替每個服務組出的識別碼，格式是「外掛 id:實例 id:服務 key（`Service.key`）」，例如 `google-services:default:google-translator`。服務預設、停用清單和翻譯歷史紀錄存的都是它（見 ADR-0003）。

**實例**（`instanceId`）：
同一個外掛裝成好幾份、各用不同帳密時用來區分的名稱。目前每個外掛只有一個實例，固定叫 `default`（`DEFAULT_INSTANCE`）。

**服務預設**（`ServicePreset`）：
一組「每個角色各用哪個服務」的選擇。可以存好幾組、隨時切換，同一時間只有一組是目前預設（`activeServicePresetId`）。介面上簡稱「預設」，但「預設」在別處也指 default，寫文件時要寫全名。
_Avoid_: 預設（單獨使用時）

**外掛狀態**（`PluginStatus`）：
外掛目前的狀態：已啟用（`ENABLED`）、已停用（`DISABLED`）、失敗（`FAILED`，載入或初始化出錯，例如 API 版本不合）、等待驗證（`AWAITING_VERIFICATION`）。

**等待驗證**（`AWAITING_VERIFICATION`）：
外掛 JAR 的 SHA-256 跟上次正常結束時記下的指紋（`PluginFingerprint`）不同，主程式先暫停這個外掛，等使用者選「接受更新」（保留外掛資料）或「全新安裝」（清掉外掛資料）。用來防止外掛被悄悄換掉。

**服務選項**（`ServiceOption`）：
服務自己宣告、讓使用者挑的參數，例如摘要長度（`summaryLength`）和改寫風格（`rewriteStyle`）。選項的詞彙由服務決定，主程式只定義幾個標準選項（`StandardOptions`）。

## 翻譯功能

**快速翻譯**（`SHOW_QUICK_TRANSLATE`）：
在任何程式選取文字後按 Ctrl+Q 跳出的小視窗，不必開主視窗就看得到譯文，是這個工具最主要的用法。
_Avoid_: 翻譯彈出視窗（設定頁的小標）

**用翻譯取代**（`REPLACE_WITH_TRANSLATION`）：
按 Ctrl+Shift+T 把選取的文字翻譯後直接貼回原處。介面標為實驗性。

**即時翻譯**（`isInstantTranslationEnabled`）：
在主視窗停止輸入一小段時間後自動翻譯，不必按翻譯。

**自動偵測**（`auto`）：
來源語言的特殊值，交給翻譯服務判斷原文是什麼語言。

**翻譯規則**（`TranslationRule`）：
「偵測到來源語言是甲就翻成乙」的對照，例如偵測到中文就翻成英文。沒有符合的規則時，用手動選的目標語言。

**固定語言**（`pinnedLanguages`）：
目標語言選單裡只列出的那幾種語言；清空就列出全部。介面上寫作「已固定語言」。

**額外輸出**（`ExtraOutputType`、`ExtraOutputSource`）：
主譯文下方的第二個結果窗格，內容可選回譯、摘要或改寫，來源可選原文或譯文。

**回譯**（`BackwardTranslate`）：
把譯文再翻回原文的語言，用來檢查譯文有沒有走樣。
_Avoid_: 反向翻譯（狀態列訊息裡的寫法）

**快速字典**（`SHOW_DICTIONARY`）：
按 Ctrl+D 跳出的浮動字典視窗，列出字詞的釋義。主視窗也可以開字典面板（`showDictionaryPanel`）；翻譯單一字詞時會自動查譯文或原文（`dictionaryAutoSource`）。
_Avoid_: 字典彈出視窗（設定頁的小標）

**圖片搜尋**（`SHOW_IMAGES`）：
按 Ctrl+Shift+Q 對選取的字詞顯示一格一格的參考圖片，給光看釋義還搞不清楚的詞用；內建來源是 Wikimedia Commons。

**文件翻譯**（`DocumentTranslationUseCase`）：
把 DOCX、PDF、TXT、SRT、VTT 整份檔案翻譯並盡量保留格式。PDF 有「保留頁面外觀」（`LAYOUT_AWARE`）和只取文字（`TEXT_ONLY`）兩種模式，前者是盡力而為。

**翻譯歷史紀錄**（`HistorySnapshot`）：
每次翻譯留下的一筆紀錄，包含原文、譯文、語言和用了哪個翻譯服務，可以在主視窗前後翻閱。

## 設定與資料

**資料目錄**（`AppDataDirectory`）：
存放設定、外掛、外掛資料和翻譯歷史紀錄的資料夾。預設就在程式旁邊；程式旁邊不能寫時才退到 `%APPDATA%\QTranslate`，也可以用 `-DappData` 指定（見 ADR-0002）。

**設定檔**（`Configuration`）：
主程式的整份設定，序列化成一段 JSON（`configuration_json`）存在資料目錄的 `datastore/` 裡。外掛自己的設定和密鑰另外存，不在這裡。

**設定版本**（`configVersion`）：
設定檔格式的版號。讀到舊版設定時，由設定遷移（`ConfigMigrator`）逐版升級到目前的格式。

**祕密儲存**（`SecretStore`）：
API 金鑰、代理密碼這類機密跟一般設定分開存放的地方：外掛的放各自那份，主程式自己的（例如代理密碼）放 `AppSecretStore`。目前沒有加密，靠資料目錄的檔案權限保護。

**網路設定**（`NetworkConfig`）：
代理伺服器、逾時與重試等連線設定，所有外掛共用同一份。

**快捷鍵**（`HotkeyBinding`）：
綁在某個動作（`HotkeyAction`）上的按鍵組合，例如快速翻譯是 Ctrl+Q。
_Avoid_: 熱鍵（工作日誌與佈署指南的寫法）

**全域快捷鍵／區域快捷鍵**（`HotkeyScope`）：
全域（`GLOBAL`）快捷鍵在任何程式裡按都會觸發，會把那組按鍵從其他程式搶走；區域（`LOCAL`）快捷鍵只在 QTranslate 視窗有焦點時才觸發。

**版面配置**（`layoutPresetId`）：
主視窗的排法，有經典（`classic`）、並排（`side_by_side`）、緊湊（`compact`）三種。介面上叫「版面配置預設」，跟服務預設無關。

**介面語言**（`interfaceLanguage`）：
程式介面顯示用的語言，跟翻譯的來源、目標語言無關。每種介面語言是 `languages/` 底下的一個 TOML 檔，例如 `zh-TW.toml`；沒設定時跟著作業系統。

**備用字型**（`editorFallbackFontConfig`）：
編輯器字型（`editorFontConfig`）缺字時拿來補字的第二個字型，只有一個名額。Windows 上沒有任何一個字型同時涵蓋中文和高棉文，所以主字型和備用字型要配成互補的一對，否則譯文會變成方框。

## 發行與 SHEICO 內部版

**發行版本**（`APP_VERSION`）：
編進程式的版本字串，顯示在「關於」和 log 開頭，也是程式內檢查更新時拿來跟 GitHub 最新版比較的數字。發行時 tag 要跟它一致。

**Windows 套件**（`WINDOWS_PACKAGE`）：
發行檔裡帶精簡 Java 執行環境的 Windows 版（`QTranslate-<版本>-windows-x64.zip`），解開就能執行，不是 MSI，也沒有安裝程式。另外兩種發行形式是要自備 Java 的可攜版（`PORTABLE`）和不含外掛的純程式 JAR（`APP_ONLY`）。

**內部版**：
SHEICO 從這個 fork 自己建置的版本，版本字串是上游版號加 `-sheico.N`，例如 `1.5.0-sheico.1`。建置分支叫 `sheico/<上游版號>-ime-fix`，推同名 tag 由 GitHub Actions 建出 Windows 套件（見 ADR-0005）。

**內部包**：
拿內部版的 Windows 套件，依公司政策拿掉部分會把文字送到外部的外掛後重新打包的 zip，放在檔案伺服器上給現場用佈署工具安裝。打包腳本、佈署工具和預先設定不在這個 repo，在另外的佈署套件資料夾。

---
🤖 opus · effort: xhigh · 2026-11-04
