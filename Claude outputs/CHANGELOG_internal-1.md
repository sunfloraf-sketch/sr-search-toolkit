# 內部開發紀錄（不上傳 GitHub）

> 此檔僅供作者本機參考，已列入 `.gitignore`，不會被 `push.bat` 上傳。
> 網站對外顯示的版本號在各頁頁尾（目前 v2.3.2）。

---

## 使用統計（GoatCounter）追蹤修正（2026-09）

**目的**：改善「感覺沒完全統計到」的問題（對應內部 C／D／E 三項）。

- **C　事件加重試**：`srCount()` 由「count.js 未載入就靜默丟棄」改為等待重試（每 250ms、最多 40 次＝10 秒），與 `countLang` 一致；避免慢網路／快手操作時的功能事件漏記。已套用 pico-builder、syntax-converter、proximity-builder、pico-builder-nonmedical。
- **D　非醫學版補追蹤**：`pico-builder-nonmedical.html` 原本完全沒有 GoatCounter。補上 count.js 腳本、`srCount`（重試版）、`countLang`（送 `lang-en`／`lang-zh`），並串到「產生策略」（`nonmed-generate`）與「下載 Word」（`nonmed-word`）兩個主要動作。
- **E　頁面路徑正規化**：五頁都在 count.js 前加入 `window.goatcounter={path:function(p){return location.pathname;}}`，使自動頁面瀏覽去掉 `?lang=en` 等 query，`/page` 與 `/page?lang=en` 不再被拆成兩筆。經模擬 count.js `get_data()` 驗證：**只作用於自動 pageview，不影響 `lang-en`／`lang-zh` 與各功能事件的顯式 path**。中英文使用統計仍由 `lang-*` 事件計算，不受影響。
- 註：E 之後即無法從路徑辨識「經英文專屬網址（`?lang=en`）進站」的人次（此為次要行銷指標）；中英使用比例請改看後台 `lang-en`／`lang-zh` 事件。
- **雙軌計數(session + no-session)**：每個事件現在送兩筆——`p`(預設,GoatCounter 依 8 小時 session 去重 ≈ 不重複使用階段/人次)與 `p-all`(帶 `no_session:true`,每次都算 ≈ 總點擊/總次數)。語言事件同理有 `lang-zh`／`lang-zh-all`。實作:`srCount` 內用 `count({...,no_session:true})`,依據 count.js 原始碼 `if(vars.no_session) data.ns=...`(送出 `ns=true`,伺服器不做 session 去重)。已套用四頁 srCount + 五頁 countLang。
  - 命名慣例(供日後自訂統計解析):去重=原名;累計=原名 + `-all`。現有事件名皆不以 `-all` 結尾,故 `endsWith('-all')` 即可判定累計版、去掉後 4 字得對應去重版。
  - 為何要驗證這件事:曾用瀏覽器實測線上頁面,確認 `pico-generate` 事件送出且 200 OK;「數字沒 +1」是 session 去重 + 後台非即時/時區,非程式問題。
- **待辦(伏筆)**：日後做一個「好讀」的自訂統計檢視(用 GoatCounter API／CSV 匯出,把每個動作的「去重 vs 累計」成對呈現,並標注實際使用量高於統計)。
- 尚未處理（需你決定）：`gc.zgo.at` 被廣告／資安過濾封鎖（在裝了擋廣告的使用者才會發生;需自訂網域代理，而 github.io 無法做，須先有自訂網域）；離線／本機 `file://` 使用天然不回報。

---

## 轉換器防呆：勾「整併為單一長式」但無組合行（2026-09）

- 轉換器：當使用者勾選「整併為單一長式」，但輸入有多行、卻**偵測不到組合行**（最後一行沒有 `#行號` 引用，例如缺少 `#1 AND #2 AND #3`）時，於結果上方顯示提醒。
- 原因：`flattenInput` 在無組合行時會「原樣返回」，導致整併其實沒作用、各檢索組分別輸出，使用者容易誤以為已整併。
- 實作：`run()` 內比對 `flattenInput(raw)===raw && 非空行數>=2`，成立則顯示 `#flatWarn`（中英雙語），每次執行先隱藏再判斷。非破壞性，不影響原輸入。
- 設計備註：PICO 產生器預設跨行以 AND 組合；轉換器則「依起始資料庫語法處理」，組合指示需使用者自行以 `#行號` 寫在最後一行。

---

## v2.3.2（2026-09）目前版本

**首頁與導覽調整**
- 首頁 PICO 由兩張卡片合併為一張「PICO 檢索策略產生器」卡片，卡內分「醫學領域」（可按，連 `pico-builder.html`）與「非醫學領域」（灰底停用、`title=整理中，暫未開放`、無連結）兩個小按鈕。
- 首頁 PICO 說明改寫：在 PICO 概念表填入自由詞彙與控制詞彙、選欄位決定完整度、一鍵生成各資料庫檢索策略；並可利用 PubMed API（內建官方輕量版）簡易測試潛在文章數量與組合，幫助發展檢索策略（中英同步）。
- 轉換器、鄰近字組合器卡片的「開啟工具 →」文字改為右下角圓形箭頭圖示（整張卡片仍可點）。
- PICO 醫學版頂端分頁的「非醫學領域」改為停用狀態（`disabled`、不可點、`整理中，暫未開放`），非只是隱藏。

**PICO 醫學版設定調整**
- Embase (Ovid)／MEDLINE (Ovid) 的「開啟資料庫」網址改為成大專用代理：`https://research.lib.ncku.edu.tw/er/geter/DB000000228`。
- 資料庫卡片標頭的「複製最終策略並開啟資料庫」按鈕移到複製圖示右側，與複製鈕並列於標頭右方。
- PubMed 輸出改為預設勾選。
- 移除 PubMed 卡片的聯絡 email 欄位：估算文章數時不再送出 email，只將檢索式送往 NCBI E-utilities；中英頁尾備註同步改為「不含 email」。

**PICO 研究設計篩選 Filter（附錄2，可複選）**
- 輸出資料庫下方新增「研究設計篩選 Filter」——**複選核取方塊**，涵蓋附錄2 全部設計：RCT（2-1）、類實驗 Quasi（2-2）、世代 Cohort／病例對照 Case-control／橫斷 Cross-sectional（2-3 拆三子項）、質性 Qualitative（2-4）、SR（2-5）、指引 Guideline（2-6）。
- 組合規則：每個「選到且該庫有語法」的設計＝獨立一行 `最終組合 AND (該庫篩選語法)`；選超過一個時，最後再加一行聯集，例：#4=組合 AND RCT、#5=組合 AND 類實驗、#6=組合 AND 質性 → #7 = #4 OR #5 OR #6。不支援的資料庫（或內容整理中的項目）略過不加行。變更選項即自動重新生成；篩選行一併進入畫面輸出與「下載 Word」。
- 篩選語法逐資料庫嵌入，原樣取自使用者紀錄表附錄（成大醫圖彙整），僅去除「Filter Source／Source／註／[完整度高]」等註解與變體標題、換行收攏為空白；Ovid／PubMed 的 RCT 採「sensitivity-maximizing（優先採用）」、Guideline 採「完整度高」變體。
- 已接語法的資料庫：RCT → Embase.com、Embase (Ovid)、MEDLINE (Ovid)、PubMed、CINAHL、WoS、Scopus；Quasi／Qualitative → Embase.com、MEDLINE (Ovid)、PubMed、CINAHL、WoS；SR → Embase.com、MEDLINE (Ovid)、PubMed、CINAHL、WoS、Scopus；Guideline → Embase.com、MEDLINE (Ovid)、CINAHL、WoS、Scopus（PubMed 附錄為「---」故未接）。
- 介面：設計選項名稱用英文（RCT／Quasi-experimental／Cohort／Case-control／Cross-sectional／Qualitative／SR／Guideline），不標「整理中／WIP」；標題與說明中英雙語：「研究設計篩選：／Study-design Filter:」（大寫 F、加冒號）、「僅支援特定資料庫／Applied only to supported databases.」（移除原「Multi-select;」）。
- PubMed 卡片頁尾備註移除「（不含 email）／(no email)」冗字（email 欄位早已刪除）。
- **待補**：2-3 的三個子項（Cohort／Case-control／Cross-sectional）目前為「顯示但停用（`disabled`、灰階不可勾）」，待使用者提供內容後填入 `STUDY_FILTERS` 並移除 disabled。Cochrane CENTRAL 專收 RCT 不套用；MEDLINE (EBSCO)、PsycInfo、華藝／國圖等暫未接。組合行沿用工具既有 `#n` 參照（非 S1/S7）。
- **篩選語法出處**：RCT、SR 的各庫出處（Cochrane Handbook Box 3.x、BMJ Best Practice、NUS、AUB、UTH libguides…）另存於 `STUDY_FILTER_SRC`，於「下載 Word」末尾以「研究設計篩選語法出處」清單列出（依設計分組、相同出處合併資料庫）。附錄 2-2／2-4／2-6 表中未附出處，故不列。

**PICO 回填檢索紀錄表（Word 就地回寫）**
- 新增「⬇ 回填檢索紀錄表」按鈕：僅在「上傳 Word」流程且已生成檢索策略時顯示（貼上流程不提供回填）；既有「下載 Word」按鈕保留。
- 就地編輯所上傳檔案的 `word/document.xml`：自動定位含「搜尋語法／Search syntax」表頭的「搜尋策略」表格，逐列讀取 `搜尋語法` 欄的提示標籤。
- **只覆蓋概念提示格**：僅當該欄提示為「X自由詞彙／X控制詞彙」時，才依「概念（首個 P/I/C/O）×自由/控制」對應到該庫生成的語法，並以**正常黑字整格覆蓋**（單一段落＋`<w:color w:val="000000"/>`，去掉原提示字）。無對應詞的概念（如僅中文的 O）維持原提示。
- **回填＝畫面所見**：把該庫在工具生成的整份檢索式（概念行＋**組合行**＋Filter 行，與畫面完全相同、含工具自身的 `#` 行號）依序黑字填入「非操作說明」的可填列；行數不足自動新增、過多移除。所有資料庫的組合行都會回填（Embase #7、Cochrane #6、Scopus #4、WoS #4…隨概念數而定）。
- **操作說明列跳過保留**：僅以「搜尋語法」欄的關鍵詞辨識操作說明列（用選單／須切換／打勾／點擊／Combine／Basic Search／Search History／Limits…）並保留不動；筆數欄的 [Ctrl+H] 等填表提示不列入辨識（否則概念列會被誤判跳過）。結果筆數欄與其餘全文原樣保留。
- 研究設計篩選（Filter）與其聯集行本就在生成結果內（`lastGen.data[].rows`），因此同一套「畫面所見」回填即涵蓋 Filter；「下載 Word」原本即已含組合行與篩選行。
- **修正：組合行被吃掉**：先前若資料庫區塊內有「合併／全寬列」（欄數 < 3，如全寬備註列或合併儲存格），該列會被當成一個填入位卻又因欄數不足被略過，導致「消耗掉一行卻沒填」——常見結果是**組合行被吞掉、只剩篩選行**。現改為「可填列」需 ≥3 欄且非操作說明列，合併／全寬列一律保留不動，組合行即正確回填（例：#5 組合、#6 #5 AND RCT、#7 #5 AND SR、#8 #6 OR #7）。
- **無搜尋策略表格的檔案**：若上傳檔找不到「搜尋語法／Search syntax」表格，改為在文件末（`sectPr` 之前）**附加**一份與「下載 Word」相同的檢索策略表（標題＋欄位完整度＋資料庫/#/搜尋語法表，含組合行、Filter 行與篩選語法出處），原文其餘內容全部保留（`appendStrategyToDoc`，以 OOXML `w:tbl` 直接建構、黑字、資料庫欄 vMerge 分組）。
- **概念表不回填、但加註**：回填不更動「檢索詞（概念表）」（保留使用者的顏色分類／縮排／灰字備註／Before 欄，這些 PICO 匯入時未保存、回填會流失）；改於概念表下方加一行紅色註記「※ 檢索詞（概念表）未由工具自動回填…請自行核對」（中英雙語，`_insertConceptNote`）。具冪等性（已加過不重覆），兩種回填情況皆加。
- 註：操作說明列內若寫死行號（如 Cochrane「#7 用選單」、Scopus「#1 AND #2 AND #3 AND #4」），因採工具行號可能與新號不一致，屬保留原文、需人工核對。
- 生成時每庫概念行存於 `RECORD_FILL[tid].slots[概念]={ft,cv}`（來源 buildForTarget 的 `label/kind` 標記）。
- 純瀏覽器、可離線：以原生 DecompressionStream／CompressionStream 讀寫 ZIP（自算 CRC32、逐檔重建中央目錄，僅置換 document.xml、其餘檔案原樣複製）、DOMParser／XMLSerializer 編修 XML；不依賴任何外部程式庫。輸出檔名為「原檔名_已回填.docx」。GoatCounter 事件 `pico-record-fill`。

**檢索詞表格：不展開狹義詞標記擴充**
- 概念表控制詞彙格的「不展開狹義詞（no-explode）」標記，除原本的 `(不含狹義詞)／(不含下位詞)` 外，新增支援 `(不狹)`、`(noexp)`、`(no exp)`（含 `(no-exp)`），以及行末裸寫的 `noexp／no exp／no-exp`。命中即改用各庫不展開語法（PubMed `[mh:noexp]`、Ovid `詞/`、Embase `/de`、Cochrane `[mh ^"詞"]`、CINAHL `MH ("詞")`）。
- 實作：擴充 `CV_NOEXP_RE` 正規式（`不狹[義詞]{0,2}`、`no[\s-]?exp\w*`、括號內或行末裸寫兩種形態，`i` 旗標）。已測不誤判（`no exposure`、`exposure` 維持展開）。填寫說明中英文範例同步補上新寫法。

**尚未整理語法之資料庫：標示「請自行確認 Filters」**
- 原「僅支援特定資料庫／Applied only to supported databases.」說明改為「尚未整理部份資料庫（平台）語法者，將於檢索策略標示『請自行確認 Filters』」（中英雙語）。
- 行為改變：先前對「所選研究設計 × 該資料庫在 `STUDY_FILTERS` 無對應語法」者是**靜默略過**；現改為在該庫檢索策略明確加註「⚠ 請自行確認 Filters（本工具尚未整理此資料庫／平台的研究設計篩選語法：<設計名>）」。三處輸出皆標示：畫面（該庫**資料庫名稱下方**、與「無控制詞彙…」等其他 ⚠ 提示並列的 `.rem` 註記，非放在策略最下方）、下載 Word（該庫表格末多一列，資料庫欄 rowspan 同步 +1）、回填檢索紀錄表（該庫區塊末多填一列）；`appendStrategyToDoc`（無策略表時）亦補該註記列。
- 設計原則：註記**不進入編號檢索式**（另存為 `exportData[].verify`＝缺語法的設計 key 陣列，非 `rows`），故「只複製語法」「執行最終策略／開啟資料庫」「PubMed 估算」等仍取正確的最後一條編號策略，複製內容不被中文註記污染。
- 新增輔助函式 `_designMiss(sels, tid)`（回傳缺語法的設計 key）與 `_verifyNote(keys)`（產生中英雙語註記字串，依 `isEn()`）。

**填寫說明改寫、貼上區說明調整、診斷回報管道、Embase (Ovid) 移除開啟連結**
- Embase (Ovid)：成大未訂購 → 由 `DB_OPEN_URLS` 移除，該庫卡片不再顯示「複製最終策略並開啟資料庫」按鈕（MEDLINE (Ovid) 仍保留）。
- 貼上診斷：移除項次的 🔧 圖示；說明改為可將結果「填寫 Google 表單回報製作者」，嵌入表單網址 `https://forms.gle/t182ArHBky8HRY3y9`，並附聯絡信箱 `flora@ncku.edu.tw`（中英雙語）。
- 「填寫說明」整段改寫，三主項標題（冒號含前字）加粗：**儲存格內：**一詞一行（預設 OR，不需標示）；**英文同義詞，請自行調整：**(1) 單字切截 `*`、(2) 移除冗贅字詞（exercise 可涵蓋 exercise therapy）、(3) 搭配鄰近字運算；**控制詞彙：**（不含狹義詞標註 → 例 `diabetes (不含狹義詞)`／`(noexp)`；單格預設 OR、特殊用 `AND`＋括號，附 Optic Nerve 範例；主標＋副標 `Optic Nerve/dg`）。
- 「貼上 AI 整理的表格」說明調整：ChatGPT 提示詞範例連結移到第一項並註「優先使用 ChatGPT 或 Claude，登入帳號效果較佳」；標題列改註「不需要的欄位可不提供；順序不拘」；補註「不同複製方式，填入效果可能有差異」；移除原「載入上次暫存」提示句。

**修正：PubMed 字尾候選勾選無效（collectVars 對 Set 用錯方法）**
- 症狀：PubMed 卡片「使用鄰近字語法」展開後，字尾候選（suffix candidates）勾選任何字形都不影響輸出。
- 原因：`collectVars()` 以 `Array.prototype.slice.call(varState[r])` 把 `Set` 轉陣列——`slice.call` 對 Set 無效（Set 非類陣列），永遠回傳 `[]`，故 `PM_VARIANTS` 的每個字根都是空陣列，`variantsOf` 一律退回原字根、不展開。轉換器用的是 `Array.from(...)`（正確）。
- 修法：比照轉換器改為 `Array.from(varState[r])`。已測：勾「diabets」後 `"gestational diabet"[tiab:~2]` 正確展開為 `… OR "gestational diabets"[tiab:~2]`。

**版面：區塊標題配色、內縮、責任聲明**
- 三個可折疊區塊標題（貼上 AI 整理的表格／填寫說明／自訂組合策略）由淺灰 `--brand2` 改為深青 `#0f766e`＋加粗，較醒目。
- 「自訂組合策略」內容比照診斷區內縮（`padding-left:1.4em`）。
- 檢索詞表右上角新增紅字責任聲明（中英）：同義詞與控制詞彙的適合性由使用者負責確認、請確實查核控制詞彙，工具僅提供詞表超連結、未進行查驗。

**修正：引號內的逗號被當成斷詞（如 'child, preschool'）**
- 症狀：自由詞欄以逗號斷詞，若某詞是帶引號的片語且引號內含逗號（如 `'child, preschool'`），會被拆成兩段。
- 修法：`toLines` 斷詞前先把引號（'、"、'、'、"、"）內的整段內容換成佔位符，斷詞後再還原，使引號內的逗號／頓號不被當成分隔。一般逗號同義詞清單（如 `gestational diabetes, GDM, diabetes in pregnancy`）與控制詞彙（`Diabetes, Gestational`）皆不受影響。已測。

**修正：Word 貼上——儲存格內含巢狀表格導致欄位錯亂**
- 症狀：從 Word 複製「檢索紀錄表」（含 Theme Concept／Before・After English Synonyms／Emtree・MeSH・CINAHL 的兩層表頭）貼上時，某格內若有 Word 的巢狀小表格（如 Before 格放 toddler*／preschool* 子表），該資料列會被灌成 8 欄、並多出一條殘列，欄位全部錯位。
- 原因：`parsePaste` 以全域 `querySelectorAll("tr")`／`tr.querySelectorAll("th,td")` 取列與儲存格，會一併抓到巢狀表格的列與儲存格。
- 修法：先選「主表格」（不被其他表格包住、儲存格最多者），只處理 `tr.closest("table")===主表格` 的列，且每列只取 `c.closest("tr")===該列` 的直接儲存格，排除巢狀表格。另 `mapCols` 的概念欄辨識加入 `Theme Concept／concept／Cut It Out`，使該類紀錄表的概念欄能對應為標籤欄。已測：P 列正確為 6 欄、標籤 P、After→自由詞、Emtree 對應、MeSH／CINAHL 留空，無殘列；含表格的一般貼上（含 Gemini）無回歸。

**修正：Gemini（iPad）貼上——控制詞彙無分隔黏詞、與「不含表格」的扁平複製**
- 症狀一（黏詞）：Gemini 複製表格時，控制詞彙（MeSH／Emtree／CINAHL）多個標目「完全無分隔」黏成一串（例：`Diabetes, GestationalPregnancy in Diabetics`），連 U+2060 都沒有。
- 修法（黏詞）：`toLines` 對控制詞彙欄（keepComma）在「小寫／數字／中日韓字 緊接 大寫」的邊界補斷詞（Title Case 標目詞內以空白分隔，故不會誤切）。已修正 MeSH、CINAHL（Title Case）；**全小寫黏詞（常見於 Emtree，如 `physical activityexercisekinesiotherapy`）無任何邊界線索，無法自動拆分**——建議改用含表格的複製、或在提示詞要求 AI 每個控制詞彙獨立一行／以分號分隔。
- 症狀二（扁平複製）：Gemini 另一種複製「不含 `<table>`」，僅為一串 `<strong>`（表頭）與帶 `data-path-to-node="0,列,欄,…"` 的 `<p>／<span>`，原解析（找不到 `<table>／<tr>`）退回純文字、欄位全亂（PICO 欄消失、MeSH／Emtree 順序被預設欄序猜錯）。
- 修法（扁平複製）：`parsePaste` 新增後備——當無 `<table>` 但 HTML 帶 `data-path-to-node` 時，依該屬性的「列,欄」重建網格（只取葉節點避免父子重複；同格多葉節點以換行併入，如標籤格 P／說明／概念名），並把最前面連續的 `<strong>` 當表頭列。僅在「無表格且含該屬性」時觸發，其餘貼上不受影響（已測含表格的正常貼上無回歸）。
- 測試：Case 1（含表格、黏詞）→ MeSH／CINAHL 正確分行、結構正確；Case 2（扁平、data-path-to-node）→ 結構完整重建（P/I…、zh/ft/mesh/emtree/cinahl 對應正確）＋MeSH／CINAHL 分行；兩例的全小寫 Emtree 仍黏在一起（已知限制）。ChatGPT／iPad（U+2060）與一般含表格貼上皆無回歸。

**修正：iPad 貼上時控制詞彙黏在一起（不可見的 U+2060 分隔）**
- 症狀：iPad（Safari）從 ChatGPT 複製表格貼上時，控制詞彙（MeSH／Emtree／CINAHL）多個詞黏成一串。
- 診斷（用內建「貼上診斷」擷取剪貼簿）：iPad 在 `text/plain` 的相鄰詞之間插入不可見的 **U+2060 WORD JOINER**（例：`Diabetes, Gestational⁠Pregnancy in Diabetics`），並非換行；當該次貼上僅有純文字（Tab 分隔列、無 `text/html`）時，`cellClean`／`toLines` 未把 U+2060 視為分隔，整格因此黏成一串。含 `text/html` 的貼上原本就靠相鄰 `<a>` 規則正確分行，不受影響。
- 修法：把 U+2060 及其他零寬不可見字元（U+2061–2064、U+200B ZWSP、U+200C ZWNJ、U+FEFF）在兩條解析路徑都當成斷詞：`cellClean`（純文字路徑）轉為換行、`_tdText`（HTML 路徑）轉為 BR；順帶清掉詞尾殘留的 U+2060。已測純文字（Tab 分隔、U+2060 黏詞）→ 正確分行；含 HTML 的貼上 → 無回歸。

**Word／回填加註「請確實查核控制詞彙」紅字，及 L3 精準 Embase /de→/mj**
- 下載 Word 與回填檢索紀錄表（含無策略表的附加流程）都在策略表下方加一行紅字提醒：「⚠ 提醒您！請尤其確實查核控制詞彙，線上工具僅提供詞表超連結，並未進行查驗。」（`_insertCVVerifyNote`，中英、具冪等性）。
- 修正：選 L3（精準）時，Embase 不展開的控制詞彙（`(不含狹義詞)` 等）由 `"詞"/de` 改對應為 `"詞"/mj`（主要焦點、不展開），與展開詞 `/exp/mj` 一致；L1／L2 維持 `/de`。新增 `CV_NOEXP_MJ`，於 `cvTermRender` 依 level 切換。**後續擴充至全部資料庫**：L3 不展開詞一律取主要焦點——PubMed `"詞"[majr:noexp]`、Ovid `*詞/`、Embase `"詞"/mj`、Cochrane `[mh ^"詞"[mj]]`、CINAHL `MM ("詞")`；L1／L2 維持各自的不展開寫法。

**貼上診斷工具（內建，供讀者自助除錯）**
- 「貼上 AI 整理的表格（提示詞範例）」區塊下方新增可折疊的「🔧 貼上診斷」（`#pasteDiagBox`）：把表格貼進診斷框（`#diagZone`）即顯示剪貼簿原始內容（`text/html`／`text/plain` 各含字元數）、解析後的欄位網格（逐列印出欄數與各格內容，格內換行以 ⏎ 標示）、以及標頭欄位對應（`mapCols` 的 label/zh/ft/mesh/emtree/cinahl 索引）。
- 重用既有 `parsePaste`／`mapCols`，與正式貼上同一套解析邏輯，故能真實重現辨識結果；「複製診斷結果」按鈕方便讀者回報作者。此區**僅供診斷、不更動 PICO 表**（已測貼上前後列數不變）。中英雙語、`#diagZone` placeholder 併入 `setLangPlaceholders`。GoatCounter 事件 `pico-paste-diag`。

**PubMed 卡片右上角自動檢索圖示（成大 LinkOut）**
- PubMed 檢索策略卡片右上角（複製圖示旁）新增「開啟後自動檢索」圖示：取最後一條組合式、遞迴展開所有 `#行號`，以 `term=` 帶入並在新分頁直接執行檢索。
- 網址改用成大專屬 LinkOut：`https://www.ncbi.nlm.nih.gov/pubmed/?otool=itwxtailib&myncbishare=nckulib&term=…`（常數 `PUBMED_NCKU_URL`）。非成大使用者亦可正常使用，僅少了成大全文連結圖示。底部原「在 PubMed 執行最終策略」按鈕同步改用此網址；估算說明列加註成大 LinkOut 提醒（tooltip 亦註明）。

**貼上表格：補回 ChatGPT 純文字欄被吃掉的換行**
- 症狀：從 ChatGPT 複製含連結的 PICO 表格貼上時，中文／英文同義詞欄整格多詞黏成一串（無法辨識為多個詞）；概念欄「標籤＋概念名」也黏在一起使標籤誤判（如 `PHIV`）。
- 診斷（用內建「貼上診斷」工具擷取剪貼簿確認）：ChatGPT 複製鈕的 `text/html` 會把「純文字欄」同格內的 `<br>` 全部移除、連空白都沒有（`HIV感染者HIV陽性個案…`），資訊已在 HTML 遺失；但 `text/plain`（Markdown 表格）仍以 `<br>` 保留每個詞的邊界。控制詞彙欄因是相鄰 `<a>` 連結，原本就已正確分行。
- 修法：新增 `_mdGrid()` 解析 `text/plain` 的 Markdown 表格（`|` 分欄、跳過 `---` 分隔列、`[詞](url)`→詞、去 `**`、`<br>`／換行→逐詞）。HTML 解析完成後若純文字表格形狀相符，逐格比對：只有當純文字切得比 HTML 更細時才用純文字覆蓋，補回換行；連帶把概念欄還原為「標籤＋概念名」兩行，標籤判讀恢復正確（`PHIV`→`P`、`O1`）。純 Markdown（無 HTML）貼上也走同一解析。
- 不影響 Word 貼上：Word 的 `text/plain` 是 Tab 分隔、無 `---` 分隔列，`_mdGrid` 回傳 null，不觸發補換行；相鄰 `<a>` 連結欄仍照舊正確。

**PICO 開啟資料庫提醒（成大代理）**
- 「複製最終策略並開啟資料庫」按鈕的提示（tooltip）第二行加註「非成大使用者，請自行開啟資料庫」（英文：Non-NCKU users…），僅套用於使用成大代理網址的目標（Embase (Ovid)／MEDLINE (Ovid)），其餘資料庫維持原提示。

**CINAHL 查核提示文字**
- CINAHL 查核浮動選單提示由「填上方『CINAHL 機構碼』」更正為「填下方」（機構碼輸入框在下方；英文 top→below）。

**PICO 控制詞彙欄標頭 hover 配色**
- 概念表 Emtree／MeSH／CINAHL 欄標頭維持原樣（彩色文字＋淺灰漸層底：Emtree 橘 `#E36C0A`、MeSH 紅 `#C0504D`、CINAHL 綠 `#00B050`）；改為「滑鼠移過去才變底色」——hover 時整格底色轉為該詞表顏色、文字轉白（淡陰影），展開查核選單（`.on`）時同樣以彩色底白字並加底線標示作用中。

## v2.3.0（2026-09）

**PICO 分為醫學／非醫學兩頁**
- 原 `pico-builder.html` 保留為醫學領域版，新增 `pico-builder-nonmedical.html`，兩頁頂端可直接切換。
- 非醫學版輸出依指定五列排列：Web of Science／Scopus；Academic Search Complete；Business Source Complete／ERIC；自訂名稱 EBSCOhost；Ei Compendex／IEEE Xplore。
- 概念表提供英文自由詞，以及 EBSCOhost、Ei Compendex、IEEE Xplore 三個「控制詞彙／原生語法」欄。原生欄可貼含欄位名稱、括號及 OR 的完整字串，系統對相符平台原樣帶入，不做跨詞表轉換。
- 支援 L1 完整／L2 平衡／L3 精準欄位、自訂 C/L 組合、行號展開、單一長式、Word 匯出、瀏覽器草稿暫存，以及「複製最終策略並開啟資料庫」。
- 依「綜合學科資料庫檢索語法大全（2025/12）」校正：EBSCOhost `N/W` 與 `S1`、WoS `TS/TI/AB/AK`、Scopus `TITLE-ABS-KEY/TITLE-ABS/AUTHKEY` 與 `W/PRE`、Engineering Village `WN KY/WN CV`、IEEE `"All Metadata"/"IEEE Terms"` 與 `NEAR/ONEAR`；同步依平台正規化布林大小寫。IEEE 不使用檢索集行號，輸出一律展開成完整式。
- IEEE Xplore 卡片新增官方限制相容性檢查：萬用字元總數與最低字元數、每子句 25 詞、欄位 OR 寫法、停用詞、引號片語及 `&` 提示；有硬性錯誤時停用推送按鈕。系統會自動大寫運算子、逐項重複欄位、展開檢索集，並只複製語法後開啟 Command Search，不自動執行或擷取結果。
- 非醫學版新增 ProQuest Dissertations & Theses Global（PQDT）：提供 PQDT 原生語法欄，依欄位完整度輸出 `noft()`、`ti()/ab()` 或 `ti()`；將無次序／有次序鄰近詞轉為 `near/n`／`pre/n`，並採小寫布林與 `[s1]` 檢索集格式。

**PubMed 初步檢索量估算**
- PubMed 卡片新增「在 PubMed 執行最終策略」按鈕：取最後一條組合式（若只有一行則取該行），展開所有 `#行號` 後，以新分頁直接開啟 PubMed 搜尋結果。
- 其他資料庫卡片新增「複製最終策略並開啟資料庫」按鈕：先展開所有 `#行號`、複製成單一完整策略，再開啟該平台官方搜尋頁。Ovid、EBSCOhost、Embase、Web of Science、Scopus 等需沿用使用者的機構登入，故不假設可跨站自動填入。
- PICO 產生器的 PubMed 卡片新增「估算 PubMed 文章數」：以 NCBI ESearch `rettype=count` 逐行查詢自由詞、MeSH 與組合式的即時命中數。
- 頁面內的 `#行號` 會先遞迴展開成完整 PubMed 檢索式，再送 API；支援預設組合、自訂 C/L 組合及整併長式。
- 請求逐筆執行並間隔 400 ms，遵守無 API key 時每秒不超過 3 次的限制（email 欄位已於 v2.3.2 移除，不送 email）。
- 查詢逾時、HTTP 或個別檢索式錯誤會標在該行，不中斷其餘估算；數量明示為查詢當下快照。
- 因此更新 PICO 頁尾與 README 的資料傳輸說明：預設仍全程本機，僅按下估算時傳送檢索式與選填 email 至 NCBI。

**新增輸出資料庫：AgeLine、ERIC（EBSCOhost）**
- 兩者為「僅輸出」目標；自由詞由工具以 EBSCO 語法自動產生（TI/AB/KW…）。
- 控制詞彙採「貼上即用」：在該庫產出的檢索策略「下方」提供逐概念貼上框，使用者貼入已寫好的展開式（如 `DE "EXERCISE" OR DE "AEROBIC exercises" OR …`），工具**原樣帶入、不改語法**，再與自由詞自動以 `OR` 組合（`#1 自由詞`／`#2 你貼的控制詞彙`／`#3 (#1 OR #2)`）。此設計不加寬概念表。預設不勾。

**控制詞彙引號重複修正**
- 若使用者在控制詞彙欄自行加了外層引號（如輸入 `"exercise therapy"`），各庫模板不再重複加引號：Embase 由 `""exercise therapy""/exp` → `"exercise therapy"/exp`；Ovid 由 `exp ""exercise therapy""/` → `exp "exercise therapy"/`（直/彎引號皆處理）。

**PICO 自動更新（已生成策略後）**
- 切換「欄位完整度」、勾選／取消「輸出資料庫」、新增／刪除／編輯檢索詞，皆自動重新生成檢索策略（編輯採 0.5 秒防抖）。未生成前不動作。

**鄰近字修正**
- 自由詞中的「鄰N單字內／以內／之內」轉換後殘留「內」字 → 已一併吸收；鄰N 仍依起始資料庫做位移（起始 Embase／Ovid／Cochrane：鄰3→NEAR/4，與鄰2→NEAR/3 一致；PubMed 等無位移：鄰3→ADJ3/NEAR3）。

**匯入判斷（PICO 上傳 Word／貼上表格）**
- 貼上表格改用與上傳 Word 一致的欄位判斷：先掃前幾列找「同時含 MeSH 與 CINAHL」的那一列當真正表頭，支援「群組列＋子欄位列」兩層表頭（如教學版檢索紀錄表）。
- 從 Word 複製整個表格貼上時，依 `colspan`／`rowspan` 展開為對齊網格：解決「主題概念」欄設 rowspan 造成表頭比資料少一欄、整列右移錯位（P/P2 跑到中文欄、Before 被當英文、Emtree/MeSH 全右移）；儲存格內多段落（如 community integration／community health nursing）也正確保留為多行。
- 自由詞一律取「After／處理後」欄；含 Before/After 時不再誤抓 Before。單一英文欄（英文同義詞／English Synonyms／Synonyms，甚至只有「概念＋英文」兩欄）也能正確匯入。
- 概念欄若因垂直合併在表頭子列留空，會自動取最左欄救回（P/I/C… 不遺失）；殘留的群組列／表頭字樣不會被當資料匯入。
- 上傳 Word 支援「整份制式檢索紀錄表」：改為掃描檔內所有表格，自動挑出檢索詞表（優先含 MeSH＋CINAHL 的表），前後有其他表格／段落也不影響；儲存格內多段落保留為多行。
- 貼上／上傳的控制詞彙：倒裝標目若在逗號後被斷成兩行（如 `Diabetes,`＋`Gestational`、`Nurses,`＋`Community Health`）會自動接回為單一詞（`Diabetes, Gestational`）。控制詞彙欄顯示維持一詞一行、不自動斷詞（`wrap=off`），過長改水平捲動。
- 貼上表格的儲存格斷行判讀重寫（修正兩個相反的問題）：只有「區塊界線（`<br>`、`</p>`、`</div>`…）」與「Unicode 換行（U+2028/2029/0085）」才算真正換行；HTML 原始碼的排版換行、縮排等 ASCII 空白一律收攏為空白。→（1）從 Word 複製貼上時，被原始碼折行的單一詞（`Diabetes,⏎Gestational`、`Pregnancy⏎in Diabetics`）不再被誤拆。
- ChatGPT 表格複製（含連結欄）黏成一串修正：ChatGPT 複製到剪貼簿的 HTML 會把同格內的多個連結詞寫成「**相鄰的 `<a>`、中間無 `<br>` 或空白**」（`<a>Diabetes, Gestational</a><a>Pregnancy in Diabetics</a>`），因此 textContent 會黏成 `Diabetes, GestationalPregnancy in Diabetics`。現在偵測「相鄰連結」自動視為各自一個詞、逐一分行；純文字欄（中文以「、」、英文以「,」分隔）維持原本正確處理。
- 上傳 Word 的控制詞彙支援兩種樣式並自動判別：（1）**分列三欄**（Emtree／MeSH／CINAHL）；（2）**舊版單欄顏色分類**——依字色分流：橘 `#E36C0A`／`#FF6600`＝Emtree、紅 `#C0504D`＝MeSH、綠 `#00B050`＝CINAHL（以色相判斷，容許色差），灰／黑字視為筆記略過。判別規則：控制詞彙內容集中在單一欄且出現目標色 → 依顏色分流；分散在多欄 → 依欄位。僅套用於 Word 上傳。

**控制詞彙欄顯示**
- MeSH／Emtree／CINAHL 欄改為「一詞一行、不自動斷詞」（`wrap=off`）：多字詞（如 Home Care Services、Attitude of Health Personnel）不再被折成兩行看似兩詞；過長改水平捲動。中文／英文同義詞欄維持自動換行。
- 概念表欄寬：英文同義詞欄 29%→22%，Emtree／MeSH／CINAHL 各 18%→21%（控制詞彙較不擠）。

**下載 Word 按鈕（深綠描金）**
- PICO `#wordBtn` 生成成品啟用時、轉換器 `.wordbtn`：改為深綠漸層＋金邊＋滑過香檳金光掃；PICO 未生成前維持既有灰色停用樣式。

**語法轉換器版面**
- 刪除「換起始庫」按鈕（多餘）；起始資料庫在左、輸出資料庫在右、上緣對齊。
- 「載入範例」移到輸入框右上，與「清空」靠右對齊；無資料時「清空」不可按。
- 輸入框加高約 1.5 行（範例 #5 組合行可完整顯示）；下載 Word 右緣對齊清空。

**頁尾備註（四頁統一）**
- 改為：「此工具（組）為純網頁程式，未使用到AI運作，僅在您的瀏覽器本機執行與暫存，不會上傳任何研究資料到網路。」末句依工具產出調整（轉換／策略實測／分堆核對／各工具輸出）；中英雙語同步。

**分析 / 後設資料**
- 新增介面語言使用統計：每次載入送出 GoatCounter 事件 `lang-en`／`lang-zh`（等分析程式載入後才送，不漏記）。
- 四頁頁尾加入 ORCID iD 作者連結（`https://orcid.org/0000-0002-7892-8840`，綠色 iD 圖示＋完整網址，新分頁開啟、`rel=noopener`），附點擊統計事件 `author-orcid`。
- 引用建議的工具名稱順序改為與導覽一致：PICO 檢索策略產生器、資料庫檢索語法轉換器、鄰近字組合器（中英同步；引用仍以 DOI 網址 `https://doi.org/…` 呈現）。

## v2.2.0（2026-09）

**台灣資料庫鄰近字**
- 華藝、國圖的鄰近字一律改用 AND（保留 OR 分組、逐詞加欄位）；交由共用引擎處理，鄰近字組合器的分組會正確帶入。
- 國圖欄位無結尾點（.ti,ab,kw）、組合行小寫 and；華藝一律併成單一條完整檢索式、去掉多餘外層括號。

**PubMed 鄰近字（PICO 與轉換器一致）**
- 預設：鄰近字改用 AND；單詞對且無切截 → 直接用 "a b"[tiab:~N]。
- 各卡片新增「使用鄰近字語法 [tiab:~N]（展開群組配對）」勾選：展開群組配對、移除切截 *、並提供字尾候選 chips。

**各庫專屬選項移到該庫卡片**
- Embase.com「排除純 MEDLINE 收錄」、華藝「納入英文片語與切截」由全域選項改為各自資料庫卡片內的勾選。
- 各資料庫的警示改顯示在該庫卡片標頭下的橘色帶（比照 PICO）；底部只留跨庫/來源層級提醒。

**版面 / 介面**
- 導覽列當頁改為實心白藥丸、他頁半透明，對比明確。
- 語法轉換器、鄰近字組合器套用與 PICO 一致的冷灰主題；轉換器清空／複製按鈕比照 PICO（ghost 灰、圖示複製）。
- 警示配色 --warn:#7B6840、--warnbg:#F3F1E6。
- 中文資料庫（華藝／國圖）輸出預設不勾。
- 轉換器「必須注意」四點取代原「語法對照表／重要注意事項」，且不用底色方框；整併長式時控制詞彙提醒改稱「含控制詞彙的區段」。
- 「片語改用鄰近字 鄰 N」依起始資料庫加位移（如起始 Embase、鄰 2 → NEAR/3）。
- 華藝提醒拆兩行；「無控制詞彙，僅用自由詞彙」；載入範例更新。

## v2.0.5（2026-09）

**新增台灣資料庫（華藝、國圖）—— PICO 產生器 + 語法轉換器**
- 新增兩個輸出資料庫：華藝線上圖書館（airiti）、臺灣博碩士論文／國圖（ndltd）。皆為「僅輸出」目標（不進起始資料庫下拉），無控制詞彙欄。
- PICO：這兩庫合併「中文同義詞(zh)＋英文同義詞(ft)」產出。
  - 華藝：英文只取單字，預設略過英文片語與含切截(*)的詞（可勾「華藝：納入英文片語與切截」加入並警示）；不支援鄰近字（維持片語）。**只有一式完成檢索** → 一律併成單一條完整檢索式（不分 #1/#2 設定行），欄位 [ALL3]（L2→[TI] OR [KW]、L3→[TI]），OR 大寫。
  - 國圖：中文加雙引號、英文可片語可切截；逐詞加欄位、以小寫 or 串接；欄位 .ti,ab,kw／.ti,kw／.ti（**無結尾點**）。仍保留 #1/#2 設定行與組合。
- 轉換器：兩庫為輸出目標；華藝鄰近字→片語、切截移除並提醒；國圖逐詞欄位。兩庫輸出加註「請補上中文同義字」。

**其他**
- Embase.com：最後一行為組合式時，預設在末尾加 `NOT (Medline/lim NOT Embase/lim)` 去重行，可用勾選取消。
- PICO：「載入範例」表格已有內容時會先跳確認；「清空」跳確認（動作無法復原）；「下載 Word」未生成前不可按；「還原上次內容」改名「載入上次暫存」，復原說明ⓘ移到表格右上角；「自訂組合策略」去齒輪並對齊；填寫說明範例區底色縮成文字寬度；起始資料庫 ⓘ 底部對齊；L3 說明措辭調整。
- 貼上 AI 表格：修正 ChatGPT 複製鈕把換行詞黏成一串（div/p/br 開頭與結尾、Unicode 換行都補斷行）。
- 版面：輸出資料庫勾選改為 3 欄緊排（WoS／Scopus 移到第三列尾端，台灣兩庫第四列）；三工具與各頁頭導覽順序統一為 PICO、語法轉換器、鄰近字。

## v2.0.4（2026-09）

**資料庫檢索語法轉換器（syntax-converter）**
- 修正「整併為單一長式」：原本只認得 `#1 內容` 一種格式，改為支援 #1／1／S1／#1.／1) 等各種行號寫法（含 Ovid 裸數字、EBSCO S1），巢狀組合行遞迴展開；`S100 protein` 之類含數字的詞不會誤判為行號。
- 範例改為「完整檢索策略」：族群(自由詞+控制詞彙) × 介入(自由詞+控制詞彙) + 組合行 `#5 (#1 OR #2) AND (#3 OR #4)`；九個起始資料庫各有對應語法的完整範例（WoS/Scopus 無控制詞彙為 2 行 + 組合）。移除範例區上方「灰字為範例預覽」說明文字。
- [註4] 與含控制詞彙行的提醒改措辭（「灰字」「控制詞彙表」；「請自行查檢此資料庫的控制詞彙表替換」）；Word 下載內的提醒同步。

**PICO 檢索策略產生器（pico-builder）**
- 移除先前試作的「WoS／Scopus 精準欄位」勾選（PICO 自由詞本就是分列精準式，該選項無實質作用）。
- PICO ↔ 鄰近字雙向來回：切到鄰近字不再丟失表格；回來（由鄰近字「送到 PICO」）時**還原原表格並把新概念接在後面**（不再重建覆蓋）。狀態保存採 sessionStorage（每分頁獨立草稿，多分頁不互蓋、F5 不掉）＋ localStorage 離開快照（供「↩ 還原上次內容」按鈕）；「清空」會一併清除暫存。新增「用鄰近字組合器加概念」按鈕。

**鄰近字組合器（proximity-builder）**
- 自動分堆規則調整：以「單一字」單獨成列者（含縮寫，如 BMD）先抓為 solo；任何「含有該 solo 字」的多字片語（forearm BMD…）視為冗贅、自動略過（solo 字本身即涵蓋）；其餘片語照原分堆。「分群檢視」會列出被略過的片語與原因。

---

## v2.0.3（2026-09）

**鄰近字組合器（proximity-builder）**
- 鄰近距離新增「鄰 0 單字內（相鄰）」。
- 新增「詞序」選項：無次序／有次序。有序僅 Embase.com（NEXT）、EBSCO（W）、Scopus（PRE）支援，其餘維持無序並加註。
- 「換行整理版」卡新增綠色「送到 PICO ＋」（可累加多概念，送出的是斷行版）；「單行 OR 版」卡為「送到轉換器」。
- 「清空」修正：連同單獨保留欄的灰字 placeholder 一併清除，詞堆保留空框。

**PICO 檢索策略產生器（pico-builder）**
- 上傳 Word 對欄修正：表頭偵測不受灰字影響、自由詞抓「After」欄、Before 不匯入、7 欄版正確對位；灰色備註字不匯入。
- 下載樣板依語言顯示（中文版／英文版各一顆）。
- 新增「整併為單一長式」勾選：把最後的 #組合行展開成一條長檢索式。
- 新增「自訂組合策略」（選填）：用概念代號 C1/C2…、以及 L1/L2… 引用前面的組合行，跨資料庫自動對應行號；概念表列首顯示代號。
- 控制詞彙進階：支援 AND/OR/NOT 與括號；次標目 `詞/dg`（Ovid 有引號）；不含下位詞 `詞(不含狹義詞)` → 各庫不展開語法；使用次標目時加註提醒；說明區加入輸入範例。
- 「語法顏色標記」預設打勾。

**資料庫檢索語法轉換器（syntax-converter）**
- 新增「整併為單一長式」勾選（在來源端展開 #行號後轉換）。

**共用引擎（兩工具）**
- 平行鏈接鄰近 `A NEAR/n B NEAR/n C`（同運算子、未加括號）不再多包一層括號。
- 巢狀鄰近偵測：只有「明確用括號把一段鄰近再當運算元」才提醒 Embase.com 不支援（中英分開顯示）。

**版面 / 介面**
- Embase.com、Embase (Ovid) 移到各版面最前（起始資料庫下拉、輸出資料庫勾選、輸出卡片）。
- 輸出資料庫勾選改為 4 行版面（Embase 兩者一行、MEDLINE 三平台一行…）。
- 語言選擇跨頁記憶（localStorage `sr_lang`），切頁不再跳回中文。
- 首頁移除上方重複的三工具導覽列；更新課程說明；引用上方加入「純網頁運算、不使用 AI、不記錄輸入」註記。

**引用 / 後設資料**
- 引用格式改為 Vancouver「Fang CJ (2026)」；中文頁面中英並列，中文作者「方靜如（2026）」；半形冒號後補空格。
- README 改為純網頁版使用（移除本機/zip 說明）；`.zenodo.json` 加入線上網址；概念 DOI `10.5281/zenodo.21960702`。

**分析**
- 加入 GoatCounter（代碼 sr-toolkit）：四頁瀏覽 + 各功能按鈕事件。

---

## 未來規劃（想做）

1. **Floating subheading 語法**
   讓次標目能以「浮動次標目（floating subheading）」正確表達，不只是黏在主標目後面。
   需處理各庫寫法，例如：Ovid `dg.fs.`（floating）／`主標目/dg`（依附）；PubMed `"diagnosis"[sh]`；Embase.com link term。目前僅支援「依附式」`詞/dg`，尚未支援真正的 floating。

2. **Subheading 跨庫對照**
   建立次標目代碼在各詞表之間的對照表（MeSH ↔ Emtree ↔ CINAHL），讓工具能自動翻譯次標目，而不是現在的「原樣帶過＋提醒」。需一份主對照表（含常用次標目與各庫代碼／全名）。

3. **API 連結測試潛在組合筆數**
   串接資料庫 API（如 PubMed E-utilities、Europe PMC、Crossref 等），對各行／各組合即時回報預估文章數，協助評估檢索策略的鬆緊。需注意：各庫 API 授權與流量限制、CORS（靜態網頁需可跨網域或走公開 API）、以及只做「估算」而非正式檢索。

4. **新增 PsycInfo、IEEE（控制詞彙）**
   加入 PsycInfo（APA Thesaurus of Psychological Index Terms）與 IEEE（IEEE Thesaurus / index terms）為輸出資料庫，含各自的控制詞彙欄與語法。並讓「只有勾選該資料庫時，才產出對應的 Word／表單欄位」（未勾選就不佔欄、不輸出），避免表單被用不到的資料庫塞滿。

---

## 待辦 / 提醒
- 發 GitHub Release 時 tag 用對應版本（如 `v2.0.3`），Zenodo 才會封存新版本。
- 次標目代碼各資料庫不通用，工具是原樣帶過 + 提醒，需人工核對。
- 更新醫學版資料庫推送入口：Embase.com 指向 `/results`，Cochrane Library 指向 `/advanced-search/search-manager`。
