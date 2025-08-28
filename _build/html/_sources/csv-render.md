---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# CSV-Rendering Script
自動將 Markdown 內的 <csv-table> 區塊轉換成可閱讀的 HTML 表格。運作流程如下：
1. 擷取 CSV 內容：讀取 <csv-table> 標籤中的純文字內容。
2. CSV 解析：使用自訂的 parseCSV 函式，逐字元解析資料，支援以雙引號包覆的欄位（例如含有逗號的文字 "New York, USA" 也能正確處理）。
3. 表格生成：將 CSV 的第一列視為標題列（<th>），其餘列轉換為資料列（<td>），組成 HTML 表格。
4. 樣式調整：表格被放入一個帶有捲軸的容器中，支援水平與垂直捲動；表格寬度會根據內容與父元素大小自動調整（至少佔滿整個容器，但不超過父元素的兩倍）。
5. 渲染輸出：最後將生成的 HTML 表格替換掉原本的 <csv-table> 區塊，讓 Markdown 文件能直接顯示出排版整齊的表格。


```javascript
document.addEventListener("DOMContentLoaded", function() {
  document.querySelectorAll("csv-table").forEach(el => {
    const csvContent = el.textContent.trim();

    // CSV 解析，支援雙引號包覆含逗號的欄位
    function parseCSV(text) {
      const rows = [];
      const lines = text.split("\n");
      for (let line of lines) {
        const row = [];
        let current = "";
        let inQuotes = false;
        for (let i = 0; i < line.length; i++) {
          const char = line[i];
          if (char === '"' && (i === 0 || line[i - 1] !== '\\')) {
            inQuotes = !inQuotes;F
          } else if (char === "," && !inQuotes) {
            row.push(current.trim().replace(/^"(.*)"$/,'$1'));
            current = "";
          } else {
            current += char;
          }
        }
        row.push(current.trim().replace(/^"(.*)"$/,'$1'));
        rows.push(row);
      }
      return rows;
    }

    const rows = parseCSV(csvContent);
    const colHeaders = rows[0];
    const htmlRows = rows.map((r, idx) => {
      if(idx === 0) return "<tr>" + colHeaders.map(c => `<th>${c}</th>`).join("") + "</tr>";
      while(r.length < colHeaders.length) r.push("");
      return "<tr>" + r.map(c => `<td>${c}</td>`).join("") + "</tr>";
    }).join("\n");

    // 建立可滾動容器
    const wrapper = document.createElement("div");
    wrapper.style.overflowX = "auto";  // 水平滾動
    wrapper.style.overflowY = "auto";  // 垂直滾動
    wrapper.style.maxHeight = "400px"; // 最大高度，可調整
    wrapper.style.marginBottom = "10px"; 
    wrapper.style.width = "100%";

    const table = document.createElement("table");
    table.border = "1";
    table.style.borderCollapse = "collapse";
    table.style.width = "max-content"; // 寬度依內容延伸
    table.style.minWidth = "100%";      // 至少撐滿容器
    table.style.textAlign = "left";
    table.innerHTML = htmlRows;

    // 限制 table 最大寬度 = 父元素寬度 * 1.5
    const parentWidth = el.parentElement ? el.parentElement.offsetWidth : 1000; // fallback
    const maxWidth = parentWidth * 2;
    table.style.maxWidth = maxWidth + "px";

    // debug log
    console.log("📐 parentWidth:", parentWidth, "→ table.maxWidth:", maxWidth);

    wrapper.appendChild(table);
    el.replaceWith(wrapper);
  });
});
```
