# CLAUDE.md — 全電商批次提報 Wireframe 專案指引

本檔案彙整本專案至今累積的設計慣例與工作流規則，供 AI 助理在後續修改此專案的 HTML 線框稿時參考，以維持風格與行為一致。

## 專案概述

這是全聯 (PX Mart) PM 團隊的「全電商」批次/預購商品提報後台管理系統的 HTML 線框稿（純前端展示用，無真實後端）。核心流程包含：申請全聯貨號（單一/批次）、商品批次提報（轉單）、草稿查詢與送審、以及各查詢結果的另開新頁面版本。

## 檔案位置與同步規則

- **G-drive（`G:\我的雲端硬碟\全聯工作PM\批次提報`）為唯一正本**，透過 Read/Write/Edit 工具直接編輯，使用者會在此資料夾中打開檔案。G-drive **無法被 bash 掛載**，只能用 Read/Write/Edit/Grep/Glob 操作。
- **outputs 資料夾**（bash 掛載路徑）僅作為驗證用途的鏡射副本，需要跑 Python HTMLParser 驗證時，才把該檔案內容 Write 一份過去，再用 bash 執行。
- G-drive 上的檔案**無法被刪除或重新命名**（系統限制）。若需要「改名」，作法是另外 Write 一個新檔案，原檔案會變成孤兒檔案（不再被任何選單連結），並告知使用者需自行手動刪除。
- 每次編輯前，尤其是相隔數個回合後，**務必重新 Read 檔案確認目前狀態，不要假設先前寫入的內容仍與記憶中一致**。本專案曾實際發生過：`批次申請全聯貨號.html` 的「勾選刪除」相關程式碼在前一輪已確認移除、驗證通過後，下一輪重新 Read 時卻發現又回到移除前的舊內容（可能是外部/使用者手動編輯或同步造成）。大範圍改動前務必先用 Grep 確認「這個修改實際影響哪些檔案、現況是否符合預期」，不要只靠記憶。

## 目前的頁面檔案清單

| 檔案 | 用途 |
|---|---|
| `批次申請全聯貨號.html` | 申請全聯貨號 > 批次申請（原檔名 `預購批次申請全聯貨號.html`，已改名，舊檔仍physically存在但無選單連結） |
| `商品單一提報.html` | 申請全聯貨號 > 單一申請全聯貨號（目前為「尚在規劃中」佔位頁，但側邊選單已套用最新規則） |
| `商品批次提報.html` | 大眾版商品管理 > 商品批次提報（轉單流程） |
| `草稿查詢與送審.html` | 大眾版商品管理 > 草稿查詢及送審（同時被上兩個批次流程的 Step5 連結，透過 `?from=` 參數判斷來源） |
| `查詢結果_預購_標籤編輯檔.html` | 批次申請全聯貨號「下載標籤編輯檔」另開新頁面版本 |
| `查詢結果_轉單_標籤編輯檔.html` | 商品批次提報「下載標籤編輯檔」另開新頁面版本 |
| `查詢結果_預購_草稿送審.html` | 批次申請全聯貨號草稿列表另開新頁面版本 |
| `查詢結果_轉單_草稿送審.html` | 商品批次提報草稿列表另開新頁面版本 |

所有 `查詢結果_*.html` 檔案**沒有側邊選單**（改用純彈窗版型：頂部只有列印/關閉視窗按鈕）。

`backup/` 資料夾內為舊版存檔，**不屬於定稿範圍**，修改/盤點/套用規則時一律排除。

## 設計 Tokens

```css
--primary:#169BD5; --pink:#DF0870; --green:#2E9E5B;
--orange:#E8912D; --red:#D0362C; --text:#333333;
--text-sub:#999999; --border:#E3E6EB; --bg:#F4F6F9;
--sidebar-bg:#1B2A4A;
```

## 側邊選單規則

- 「大眾版商品管理」為可收合群組，**預設收合**（`display:none`），唯獨在 `商品批次提報.html` 中因該頁功能本身屬於此群組，預設展開且該項目標記為 active。
- 「申請全聯貨號」群組**不包含**「商品批次提報」（該功能屬於大眾版商品管理）；「商品單一提報」已改名為「單一申請全聯貨號」。
- 收合群組的標準寫法：

```html
<div class="nav-item group" onclick="toggleNavGroup('groupId', this)">
  群組名稱<span class="chevron">▾</span>
</div>
<div class="nav-children" id="groupId" style="display:none;">
  <!-- 子項目 -->
</div>
```

```css
.nav-item.group .chevron{margin-left:auto;font-size:11px;transition:transform .2s;}
.nav-item.group.expanded .chevron{transform:rotate(180deg);}
.nav-children{overflow:hidden;}
```

```js
function toggleNavGroup(id, headerEl){
  var el = document.getElementById(id);
  var isOpen = el.style.display !== 'none';
  el.style.display = isOpen ? 'none' : 'block';
  headerEl.classList.toggle('expanded', !isOpen);
}
```

預設展開的變體：`class="nav-item group expanded"` + `.nav-children` 不加 inline `style="display:none;"`。

## 草稿查詢與送審的來源判斷

`草稿查詢與送審.html` 同時被 `商品批次提報.html`（`?from=batch`）與 `批次申請全聯貨號.html`（`?from=preorder`）連結，需依據 URL 參數動態改寫麵包屑與側邊選單 active 狀態（IIFE `applyOriginContext`，用 `URLSearchParams(location.search).get('from')` 判斷）。`商品單一提報.html` 的兩個選單入口（大眾版商品管理群組內、申請全聯貨號群組內）皆指向同一支檔案，目前僅是靜態佔位頁，未套用 `?from=` 動態邏輯。

## 全聯分類 Mock 資料只能使用真實分類

Mock 商品的「全聯分類」欄位（無論是表格內的資料，或是查詢條件 filter-bar 的下拉選單 `<option>`）**一律只能使用以下真實 PX Mart 分類路徑**（來源：`全聯分類滙出_20250905.xlsx`），不可再自創「飾品配件&gt;耳環&gt;耳針式&gt;925純銀」之類的虛構分類：

- `服飾配件/鞋包&gt;流行飾品/包類/傘類&gt;金飾/珠寶&gt;金/銀飾類_77010501`（涵蓋所有戒指/項鍊/耳環/手鍊等飾品，材質不拘）
- `服飾配件/鞋包&gt;流行飾品/包類/傘類&gt;髮飾&gt;髮束_77010403`
- `服飾配件/鞋包&gt;流行飾品/包類/傘類&gt;髮飾&gt;邊夾_77010401`

日後新增 mock 列或 filter-bar 選項時，一律從這 3 個之中挑選，不要沿用舊的虛構分類字串（表格資料與 filter-bar 選單兩處都要檢查，曾發生只改表格資料、漏改 filter-bar 選單的情況）。

## 「標籤母表」雙顯示模式（重要）

「下載標籤編輯檔」查詢結果表格新增「標籤母表」欄位，**欄位順序固定在「標籤」狀態欄之後、「草稿編號」之前**（不是放在「全聯分類」後面）。

**filter-bar 查詢條件也要有對應的「標籤母表」下拉選單**，選項為真實母表清單中的名稱（來源：使用者提供的 `標籤母表清單.xlsx`），示範用 5 筆：

```html
<div class="fld"><label>標籤母表</label>
  <select style="min-width:200px;">
    <option>全部母表</option>
    <option>飾品</option>
    <option>髮飾</option>
    <option>包包</option>
    <option>餐具</option>
    <option>鍋具/鍋具配件</option>
  </select>
</div>
```

表格內「標籤母表」儲存格的文字，**只能是上述下拉選單清單中出現過的母表名稱本身**（例如「飾品」「髮飾」），**不可自創編號**（曾誤用 `MS-2026-011` 這類流水號，已全部改回真實母表名稱）。

顯示邏輯（依「標籤」狀態 + 全聯分類 綜合判斷，兩條規則都要套用）：

1. **狀態為「通過」的列 → 一律純文字**，不論分類為何（該列的母表已確定，不需再選）。
2. **狀態為「編輯」或「未填」的列 → 依分類決定**：
   - 分類為「金/銀飾類」（該分類有 1 張以上母表可用）→ 呈現 `<select>` 下拉選單，預設選第 1 張：
     ```html
     <td><select style="padding:2px 4px;font-size:12px;"><option selected>飾品</option><option>包包</option></select></td>
     ```
   - 分類為「髮束」或「邊夾」（該分類只有 1 張母表可用）→ 呈現純文字：
     ```html
     <td>髮飾</td>
     ```

範例對照（`商品批次提報.html` main-3）：通過列一律文字；編輯/未填列中屬「金/銀飾類」的顯示「飾品」下拉，屬「髮束/邊夾」的顯示「髮飾」文字。

## 「標籤」狀態新增「未填」值

「標籤」狀態欄除了既有的「通過」（綠色 `.cell-link.pass`）、「編輯」（粉色 `.cell-link.edit`）之外，新增第三種「未填」（橘色，代表尚未建立標籤）：

```css
.cell-link.unfilled{color:var(--orange);}
```

```html
<span class="cell-link unfilled">未填</span>
```

`check-legend` 說明文案需一併補上：`「未填」= 尚未建立標籤`。目前僅 `商品批次提報.html` main-3 的 mock 資料中有實際「未填」列，其餘檔案的資料集尚未套用（因為套用「標籤母表」雙顯示規則時，「未填」列的呈現規則與「編輯」列相同，都是「依分類決定」，所以就算某檔案資料集裡沒有「未填」列，只要規則本身有套用即可）。

## 刪除/複製按鈕規範（已改版：移除批次「勾選刪除」機制）

**重要變更**：本專案原本每個表格都有「勾選刪除」批次刪除按鈕 + 對應的 `.del-check` checkbox 欄（表格最左或最右一欄）。**此機制已依需求全面移除**，包括：
- 按鈕本身（`<button id="xxxDelBtn">` 或 `<div id="xxxDelBtn" class="ghost-btn is-disabled">`）
- 表頭的全選 checkbox `<th><input type="checkbox" ...title="全選刪除"></th>`
- 每一列的 `<td><input type="checkbox" class="del-check"></td>`
- script 內 `_bulkDeleteBtns` 陣列中對應的 DelBtn 項目（**下載按鈕**的 `dl-check` 項目要保留，只移除跟 `.del-check`/「勾選刪除」有關的項目）

現況（已套用到全部 7 個非 backup 檔案 + 4 個 `查詢結果_*.html` 彈窗）：
- 「下載標籤編輯檔」類表格（`商品批次提報.html` main-3、`批次申請全聯貨號.html` panel-3、及其 2 個彈窗）：僅保留單列「刪除」按鈕（`<span class="small-btn red" onclick="showDeleteConfirm(...)">刪除</span>`），以及左側 `.dl-check` 下載勾選欄（維持不變）。
- 「草稿查詢與送審」類表格（`草稿查詢與送審.html`、`商品批次提報.html` main-5、及其 2 個彈窗 `查詢結果_*_草稿送審.html`）：同樣移除批次刪除機制。**`草稿查詢與送審.html` 與其兩個彈窗版本另外新增了「複製」按鈕**，放在「單筆刪除」的左邊，同一個 `<td>` 內：

```html
<td>
  <span class="small-btn blue" onclick="copyDraftRow(this)">複製</span>
  <span class="small-btn red" onclick="showDeleteConfirm('full',1,function(row){row.remove();}.bind(null,this.closest('tr')))">單筆刪除</span>
</td>
```

```css
.small-btn.blue{border-color:var(--primary);color:var(--primary);}
```

```js
function copyDraftRow(el){
  var tr = el.closest('tr');
  var clone = tr.cloneNode(true);
  tr.parentNode.insertBefore(clone, tr.nextSibling);
}
```

**下載按鈕（`.dl-check` / `dl-check`）的規則沒有變**，仍依原規則運作：表格最左欄選取列 checkbox 需加 `class="dl-check"`、獨立控制下載按鈕 enable/disable，跟刪除機制的 `.del-check` 完全脫鉤（del-check 現在已不存在於表格 UI 中，`refreshBulkDeleteBtn` 函式本體與 `checkClass = checkClass || 'del-check'` 這行 fallback 保留在 script 內是正常的，屬於共用工具函式的預設值，不是 bug，不用特地清除）：

```js
function refreshBulkDeleteBtn(scope, btnId, checkClass){
  checkClass = checkClass || 'del-check';
  var checks = document.querySelectorAll((scope ? scope + ' ' : '') + '.' + checkClass);
  var any = Array.prototype.some.call(checks, function(cb){ return cb.checked; });
  var btn = document.getElementById(btnId);
  if (!btn) return;
  if (btn.tagName === 'BUTTON') btn.disabled = !any;
  btn.classList.toggle('is-disabled', !any);
}
var _bulkDeleteBtns = [
  { scope: '#panel-3', id: 'panel3DownloadBtn', checkClass: 'dl-check' }
  // 每個檔案依自己的下載按鈕 id 增列；不要再加 DelBtn 項目
];
```

```css
.ghost-btn:disabled{opacity:.45;cursor:not-allowed;}
.ghost-btn.is-disabled{opacity:.45;cursor:not-allowed;pointer-events:none;}
```

## 「下載標籤編輯檔」區塊版面規範（result-count + toolbar-v2）

**示範標準：`批次申請全聯貨號.html` 的 panel-3 / `商品批次提報.html` 的 main-3。**

```html
<div class="result-count" style="justify-content:flex-start;">
  <span>查詢結果（共 <b>20</b> 筆）</span>
  <div class="check-legend" style="margin-top:0;">「通過」= 標籤已檢測合格　「編輯」= 尚未通過，需點擊修正並勾選後下載標籤編輯檔　「未填」= 尚未建立標籤</div>
</div>
<div class="toolbar-v2">
  <div class="btn-row-left" style="align-items:center;margin-bottom:0;">
    <button id="xxxDownloadBtn" class="primary-btn" disabled>⬇ 下載標籤編輯檔</button>
    <!-- 只有「有側邊選單的頁面」才有這顆，查詢結果_*.html 彈窗版沒有 -->
    <button class="open-newpage-btn" style="padding:8px 18px;font-size:14px;font-weight:700;" onclick="window.open('查詢結果_xxx.html','_blank')">↗ 查詢結果另開新頁面</button>
  </div>
</div>
```

規則：
1. 說明文案（`「通過」=...「編輯」=...「未填」=...`）放在「查詢結果（共 N 筆）」的 `.result-count` 列尾端，不放在 toolbar 內。
2. 「↗ 查詢結果另開新頁面」按鈕（若該頁有）放在「下載標籤編輯檔」按鈕右邊、同一個 `.btn-row-left`；樣式維持白底藍框（`.open-newpage-btn` 原本的 class 不變），但 size 用 inline style 對齊 `.primary-btn`（`padding:8px 18px;font-size:14px;font-weight:700;`）。
3. **toolbar-v2 內不再需要 `justify-content:space-between` 的外層 flex div**（那是舊版為了把「勾選刪除」推到最右邊才加的，勾選刪除已移除，`.btn-row-left` 直接放在 `.toolbar-v2` 底下即可）。

## 查詢條件與查詢結果的區塊分界

有 filter-bar（查詢條件）+ 查詢/清除按鈕的頁面，在「查詢/清除」按鈕列與「查詢結果」列之間，一律插入一條分界線，讓兩個區塊有視覺區隔：

```html
<hr style="border:none;border-top:1px solid var(--border);margin:14px 0;">
```

`查詢結果_*.html` 彈窗頁本身沒有查詢條件（只顯示結果），不適用此規則。

## 分頁預設筆數

所有查詢結果的「每頁顯示」下拉選單，20 需明確標示為預設值（避免僅依賴瀏覽器「預設選第一個 option」的隱性行為）：

```html
<select><option selected>20</option><option>50</option><option>100</option><option>500</option></select>
```

## Mock 商品名稱長度需貼近真實

商品名稱欄位使用的 mock 資料，長度需落在 30~50 字之間（模擬真實電商長標題，而非「925純銀珍珠耳針」這種過短的示意文字）。同一 SKU 出現在多個檔案時（見下方對照表），名稱需保持一致，不要各自為政。

## 草稿編號（草稿查詢與送審 / 下載標籤編輯檔 兩套表格共用同一批 mock 商品）

「下載標籤編輯檔」查詢結果表格與「草稿查詢與送審」草稿列表，背後是同一批 mock 商品資料，因此「草稿編號」欄位（放在「標籤母表」欄位之後、「商品名稱」欄位之前）必須用**規格型號 (SKU)** 去對應，而不是用列順序（兩邊表格的列順序不一定相同）。目前已知對應：

- 批次申請全聯貨號.html / 查詢結果_預購_標籤編輯檔.html：ER-PL925-01→100032、NK-VT-02→100033、BR-GEO-03→100034、RG-SET-04→100035
- 商品批次提報.html / 查詢結果_轉單_標籤編輯檔.html：HR-CMP-02→100042、BR-SET-11→100041、ER-CMP-05→100043、NK-SET-07→100044

日後新增類似對照欄位時，先用 SKU grep 兩邊表格確認對應關係，不要直接依序複製編號。

## HTML 驗證方式

每次修改後，若要跑自動化標籤平衡驗證，先把該檔案內容 Write 一份到 outputs 對應資料夾，再於 bash 用以下 Python HTMLParser 腳本檢查標籤是否平衡（`unclosed` 與 `errors` 皆須為空陣列才算過關）。若無法跑 bash（例如本輪不需要或使用者尚未同意），可改用 Grep/Read 人工核對每個 `<thead>` 的 `<th>` 數與每個 `<tbody><tr>` 的 `<td>` 數是否一致，以及 `<div>`/`</div>`、`<tr>`/`</tr>` 等標籤數量是否配對，作為替代驗證手段。

```python
from html.parser import HTMLParser
class V(HTMLParser):
    VOID={'br','img','input','meta','hr','link'}
    def __init__(s):
        super().__init__(); s.stack=[]; s.errors=[]
    def handle_starttag(s,tag,attrs):
        if tag in s.VOID: return
        s.stack.append(tag)
    def handle_endtag(s,tag):
        if not s.stack:
            s.errors.append(('extra close',tag)); return
        if s.stack[-1]==tag:
            s.stack.pop()
        else:
            if tag in s.stack:
                while s.stack and s.stack[-1]!=tag:
                    s.errors.append(('mismatch',s.stack.pop()))
                if s.stack: s.stack.pop()
            else:
                s.errors.append(('mismatch-notfound',tag))
# 對每個檔案 feed 內容後印出 v.stack 與 v.errors
```

## 一般工作習慣

- 使用者習慣一次丟出多個指令（有時中間夾雜新訊息），修改範圍經常橫跨 6~8 個檔案，務必用 grep 先確認「這個修改實際影響哪些檔案」，避免漏改或誤改。
- 每次大規模改動後，主動跟使用者說明「這次改了哪幾個檔案、還有哪些檔案尚未套用」，不要靜默留下不一致的狀態。
- 使用者偏好精簡直接的回覆，不需要條列每一步驟的過程說明。
- 當使用者說「以 XXX.html 為範本，更新 YYY.html」時，代表要把 XXX.html 目前**實際的 HTML 內容**（不是文件裡記的規則）原封不動地在 YYY.html 對應區塊套用一次，包含資料呈現邏輯與 class/樣式；套用前務必重新 Read 兩份檔案的最新狀態（尤其 YYY.html，因為曾發生檔案被還原成舊版的狀況）。
