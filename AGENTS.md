# AGENTS.md

## Project overview

Static HTML wireframe prototype for PX Mart (全聯) PXEC's "全電商" batch/preorder product-reporting admin UI. No backend, no build step — every page is a single self-contained `.html` file (inline `<style>` + `<script>`). Pages: 貨號申請 (single/batch), 商品批次提報 (轉單), 草稿查詢與送審, and their no-sidebar "另開新頁面" popup variants (`查詢結果_*.html`).

## File locations

- Source of truth: `G:\我的雲端硬碟\全聯工作PM\批次提報\` — edit here directly. This folder is **not bash-mountable**; only the Read/Write/Edit/Grep/Glob file tools reach it.
- Validation mirror: when a Python HTMLParser check is needed, Write the current file content into the bash-mounted `outputs/` copy first, then run the validator there via bash.
- G-drive files cannot be deleted or renamed. To "rename" a file, write a new file with the new name and leave the old one orphaned (unlinked from navigation); tell the user to delete the stale file manually.
- Before editing a file that hasn't been touched this session, **re-read it first — do not assume it still matches what was last written.** This has bitten us for real: `批次申請全聯貨號.html` had its "勾選刪除" (bulk-delete) code fully removed and verified clean in one turn, then reappeared with the old code on the very next Read in a later turn (likely an external edit or sync). Re-verify with Grep before trusting memory, especially before large multi-file batch edits.
- The `backup/` folder holds superseded copies — always exclude it from "current/final" file surveys, bulk edits, or convention audits.

## Setup

No install step. To validate HTML, run the Python snippet in "Validation" below via the bash tool, `cd`'d into the outputs mirror directory (after mirroring the file there — see above).

## Conventions

### Design tokens
`--primary:#169BD5 --pink:#DF0870 --green:#2E9E5B --orange:#E8912D --red:#D0362C --text:#333333 --text-sub:#999999 --border:#E3E6EB --bg:#F4F6F9 --sidebar-bg:#1B2A4A`

### Sidebar
- "大眾版商品管理" group is collapsible, default collapsed, except in `商品批次提報.html` where it's default expanded (that page's function lives inside this group).
- "申請全聯貨號" group does not include "商品批次提報" (that lives only in 大眾版商品管理). "商品單一提報" is labeled "單一申請全聯貨號" wherever it's linked from the 申請全聯貨號 group.
- Collapsible group pattern:
  ```html
  <div class="nav-item group" onclick="toggleNavGroup('id', this)">Label<span class="chevron">▾</span></div>
  <div class="nav-children" id="id" style="display:none;">...</div>
  ```
  ```js
  function toggleNavGroup(id, headerEl){
    var el = document.getElementById(id);
    var isOpen = el.style.display !== 'none';
    el.style.display = isOpen ? 'none' : 'block';
    headerEl.classList.toggle('expanded', !isOpen);
  }
  ```
- All `查詢結果_*.html` files have no sidebar at all (popup template: topbar with only 列印/關閉視窗).
- `草稿查詢與送審.html` is shared by two entry points; it reads `?from=batch|preorder` from the URL to pick the correct breadcrumb text and sidebar active item. `商品單一提報.html` (currently a "尚在規劃中" placeholder page) is linked from both groups too, but does not yet implement `?from=` context switching — its sidebar just marks the 申請全聯貨號 > 單一申請全聯貨號 entry active statically.

### Mock 全聯分類 values must use the real taxonomy

Every mock "全聯分類" value — both table cell data and filter-bar `<select>` options — must be one of these three real PX Mart category paths (source: `全聯分類滙出_20250905.xlsx`). Do not invent fictional categories like `飾品配件>耳環>耳針式>925純銀`:

- `服飾配件/鞋包>流行飾品/包類/傘類>金飾/珠寶>金/銀飾類_77010501` (covers all rings/necklaces/earrings/bracelets regardless of material)
- `服飾配件/鞋包>流行飾品/包類/傘類>髮飾>髮束_77010403`
- `服飾配件/鞋包>流行飾品/包類/傘類>髮飾>邊夾_77010401`

When adding new mock rows or filter options, pick from these three only. Check both the table data AND the filter-bar dropdown — it's easy to update one and forget the other.

### "標籤母表" (label master-table) dual-display column

The "下載標籤編輯檔" result tables have a "標籤母表" column, positioned **right after the 標籤 status column, before 草稿編號** (not after 全聯分類).

The filter-bar needs a matching "標籤母表" dropdown, using real master-table names (source: user-provided `標籤母表清單.xlsx`), 5 sample entries:
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

Cell content must be one of those exact master-table names — never an invented code like `MS-2026-011` (this was tried and explicitly reverted).

Display logic (status + category both matter):
1. Status = "通過" (pass) → **always plain text**, regardless of category (already finalized, nothing left to pick).
2. Status = "編輯" (edit) or "未填" (unfilled) → depends on category:
   - "金/銀飾類" (multiple master tables available) → `<select>` dropdown, first option selected:
     ```html
     <td><select style="padding:2px 4px;font-size:12px;"><option selected>飾品</option><option>包包</option></select></td>
     ```
   - "髮束" / "邊夾" (only one master table available) → plain text:
     ```html
     <td>髮飾</td>
     ```

### "標籤" status: new "未填" value

Third status value alongside 通過 (green, `.cell-link.pass`) and 編輯 (pink, `.cell-link.edit`): 未填 (orange), meaning the label hasn't been created yet.
```css
.cell-link.unfilled{color:var(--orange);}
```
```html
<span class="cell-link unfilled">未填</span>
```
Update `check-legend` text to add: `「未填」= 尚未建立標籤`. Currently only `商品批次提報.html` main-3's mock data has actual 未填 rows; other files' datasets happen not to include any, but the rendering rule (see above) is the same regardless — 未填 follows the same category-based dual-display logic as 編輯.

### Delete / copy buttons (redesigned — bulk "勾選刪除" mechanism removed)

**Breaking change from earlier in the project**: every table used to have a bulk "勾選刪除" button plus a `.del-check` checkbox column (header select-all `<th>` + one `<td>` per row). **This has been removed entirely**, across all 7 non-backup files plus the 4 `查詢結果_*.html` popups:
- the button element (`<button id="xxxDelBtn">` or `<div id="xxxDelBtn" class="ghost-btn is-disabled">`)
- the header select-all checkbox `<th><input type="checkbox" ... title="全選刪除"></th>`
- each row's `<td><input type="checkbox" class="del-check"></td>`
- the corresponding entry in that file's `_bulkDeleteBtns` script array (keep the **download button**'s `dl-check` entry — only remove the delete-button entries)

Current state:
- "下載標籤編輯檔" tables (`商品批次提報.html` main-3, `批次申請全聯貨號.html` panel-3, and their 2 popups): only a single-row `刪除` action remains (`<span class="small-btn red" onclick="showDeleteConfirm(...)">刪除</span>`); the left-side `.dl-check` download-selection column is untouched.
- "草稿查詢與送審" tables (`草稿查詢與送審.html`, `商品批次提報.html` main-5, and its 2 popups `查詢結果_*_草稿送審.html`): same bulk-delete removal. **`草稿查詢與送審.html` and its 2 popups additionally got a new "複製" (copy) button**, placed to the left of "單筆刪除" in the same `<td>`:

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

The download button (`.dl-check`) mechanism is unchanged: the leftmost row-select checkbox column still needs `class="dl-check"` and drives the download button's enable/disable state independently of the (now-removed) delete mechanism. Leaving `refreshBulkDeleteBtn`'s `checkClass = checkClass || 'del-check'` fallback line and its `.del-check` query-selector string in the shared helper function is fine — that's just unused generic default-parameter plumbing, not a bug, and doesn't need to be cleaned out:

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
  // add one entry per download button in this file; do not add DelBtn entries anymore
];
```
```css
.ghost-btn:disabled{opacity:.45;cursor:not-allowed;}
.ghost-btn.is-disabled{opacity:.45;cursor:not-allowed;pointer-events:none;}
```

### "下載標籤編輯檔" block layout — reference is `批次申請全聯貨號.html` panel-3 / `商品批次提報.html` main-3

```html
<div class="result-count" style="justify-content:flex-start;">
  <span>查詢結果（共 <b>20</b> 筆）</span>
  <div class="check-legend" style="margin-top:0;">「通過」= 標籤已檢測合格　「編輯」= 尚未通過，需點擊修正並勾選後下載標籤編輯檔　「未填」= 尚未建立標籤</div>
</div>
<div class="toolbar-v2">
  <div class="btn-row-left" style="align-items:center;margin-bottom:0;">
    <button id="xxxDownloadBtn" class="primary-btn" disabled>⬇ 下載標籤編輯檔</button>
    <!-- only on pages that have a sidebar; the 查詢結果_*.html popup variants omit this -->
    <button class="open-newpage-btn" style="padding:8px 18px;font-size:14px;font-weight:700;" onclick="window.open('查詢結果_xxx.html','_blank')">↗ 查詢結果另開新頁面</button>
  </div>
</div>
```
- The description text (`「通過」=...「編輯」=...「未填」=...`) sits in the `.result-count` row (after "查詢結果（共 N 筆）"), not inside the toolbar.
- "↗ 查詢結果另開新頁面" (when present) sits beside the download button, same `.btn-row-left`; keep its white-bg/blue-border class but size it to match `.primary-btn` via inline style.
- **No outer `justify-content:space-between` flex wrapper div is needed anymore inside `.toolbar-v2`** — that was only there to push the (now-removed) "勾選刪除" button to the far right. `.btn-row-left` sits directly inside `.toolbar-v2`.

### Query conditions vs. results — visual separation
On pages with a filter-bar (query conditions) plus 查詢/清除 buttons, insert a divider between the button row and the result section:
```html
<hr style="border:none;border-top:1px solid var(--border);margin:14px 0;">
```
The `查詢結果_*.html` popups have no query conditions (results only) and don't need this.

### Pagination default
Every "每頁顯示" page-size `<select>` must mark 20 as explicitly selected, not rely on the browser defaulting to the first `<option>`:
```html
<select><option selected>20</option><option>50</option><option>100</option><option>500</option></select>
```

### Mock product name realism
商品名稱 mock values should be 30-50 characters long (realistic e-commerce-style titles), not short placeholders. The same SKU appearing in multiple files (see mapping below) must use the same name everywhere — don't let copies drift.

### 草稿編號 cross-referencing
The "下載標籤編輯檔" result tables and the "草稿查詢與送審" draft tables share the same mock products. The 草稿編號 column (placed right after 標籤母表, before 商品名稱) must be matched by SKU (規格型號), not row position — row order differs between the two tables. Known mappings:
- 批次申請全聯貨號.html / 查詢結果_預購_標籤編輯檔.html: ER-PL925-01→100032, NK-VT-02→100033, BR-GEO-03→100034, RG-SET-04→100035
- 商品批次提報.html / 查詢結果_轉單_標籤編輯檔.html: HR-CMP-02→100042, BR-SET-11→100041, ER-CMP-05→100043, NK-SET-07→100044

## Validation

Preferred: mirror the changed file's content into the bash-mounted `outputs/` copy, then run the Python HTMLParser snippet below against it.

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
# feed each changed file's contents into a V() instance;
# it must report stack == [] and errors == [] before the change is considered done.
```

If bash mirroring isn't available or appropriate for the current turn, fall back to a manual check via Read/Grep: confirm each `<thead>` row's `<th>` count matches the `<td>` count of every `<tr>` in the corresponding `<tbody>`, and that `<div>`/`</div>`, `<tr>`/`</tr>`, `<table>`/`</table>` etc. counts balance.

There is no other test suite. A change is "done" only when the validator (or the manual equivalent) reports clean for every file touched.

## PR / handoff instructions

There's no PR flow — this is a single-user wireframe project edited live. After a batch of edits, tell the user (concisely) which files were changed and flag any file that was in scope but not yet updated, rather than leaving an inconsistent state undisclosed. When the user says "use X.html as the template, update Y.html", replicate X.html's **actual current HTML** (not just the rules documented here) into the corresponding section of Y.html — re-read both files fresh first, since files (especially ones not touched recently) can drift from their last-known state.
