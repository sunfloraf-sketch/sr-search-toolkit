# SR Search Toolkit · 系統性文獻回顧檢索工具組

<!-- After you connect Zenodo and publish a release, replace the line below with the DOI badge Zenodo gives you. -->
<!-- 連結 Zenodo、發布 release 後，把下面這行換成 Zenodo 給你的 DOI 徽章。 -->
[![DOI](https://zenodo.org/badge/DOI/PLACEHOLDER.svg)](https://doi.org/PLACEHOLDER)

Three independent, browser-based tools for systematic-review (SR) literature searching. Each runs entirely in your browser — nothing is uploaded — and every page links to the other two.

三個獨立、在瀏覽器中執行的系統性文獻回顧（SR）檢索小工具。全部在本機瀏覽器執行、不上傳任何資料，每一頁都能跳到另外兩個工具。

## Tools · 工具

The three tools are **not sequential** — use whichever you need.

三個工具**沒有先後順序**，依需要挑選使用。

| Tool | 工具 | What it does |
|------|------|--------------|
| **Proximity Builder** (`proximity-builder.html`) | 鄰近字組合器 | Turn stacks of synonyms into a proximity (adj/near/N) fragment, or auto-split a *Before* list into stacks. Can send the result straight to the Converter. |
| **PICO Strategy Builder** (`pico-builder.html`) | PICO 檢索策略產生器 | Fill in free-text and controlled terms per PICO concept, pick a field level, and generate line-numbered strategies for each database, with MeSH / Emtree / CINAHL Headings mapping. |
| **Search Syntax Converter** (`syntax-converter.html`) | 資料庫檢索語法轉換器 | Paste a query written for any starting database and generate syntax for seven databases/platforms in one click; reference lines with `#1 #2`, and export to Word. |

Supported databases/platforms in the Converter: MEDLINE (Ovid), PubMed, Cochrane, Embase, CINAHL (EBSCOhost), Web of Science, Scopus.

## Usage · 使用方式

**Locally 本機**: open `index.html` in any modern browser (Chrome, Edge, Firefox, Safari). Keep all four HTML files in the same folder so the cross-links and data hand-off work.

本機使用：用瀏覽器打開 `index.html` 即可。請把四個 HTML 檔放在同一個資料夾，互連與資料帶入才會正常。

**Online 線上**: if you enable GitHub Pages for this repository (Settings → Pages → Deploy from branch → `main` / root), the toolkit is served at:

線上使用：在此 repo 開啟 GitHub Pages（Settings → Pages → Deploy from branch → `main` / root），即可在以下網址使用：

```
https://sunfloraf-sketch.github.io/sr-search-toolkit/
```

Both a Chinese and an English interface are available — use the language button in the top-right corner of any page.

介面提供中文與英文，點各頁右上角的語言按鈕切換。

## Citation · 引用

If you use this toolkit in your work, please cite it. After publishing a release on Zenodo you will get a permanent DOI; cite that DOI, or use the metadata in [`CITATION.cff`](CITATION.cff).

如果你在研究中使用本工具，請引用。在 Zenodo 發布 release 後會取得永久 DOI，請引用該 DOI，或使用 [`CITATION.cff`](CITATION.cff) 內的資訊。

> Fang, C.-J. (2026). *SR Search Toolkit: Proximity Builder, PICO Strategy Builder, and Search Syntax Converter* (Version 1.0.0) [Software]. https://doi.org/PLACEHOLDER

## License · 授權

Released under the [MIT License](LICENSE) — free to use, modify, and redistribute, provided the copyright notice is retained.

以 [MIT 授權](LICENSE) 釋出——可自由使用、修改、再散布，保留著作權聲明即可。

© 2026 Ching-Ju Fang
