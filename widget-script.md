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

# Decap CMS Widget Script
## CSV Widget 核心邏輯說明
1. Widget 註冊
  - CMS.registerEditorComponent({...})：在 Decap CMS 註冊一個新的編輯器元件，名稱為 csv-table。
2. 欄位
  - fields：定義編輯介面需要顯示的輸入欄位。
  - pattern：透過正規表示式比對。
3. 轉換
  - fromBlock()：將 Markdown 內的 &lt;csv-table&gt; 區塊，轉換成內部資料物件，方便在編輯器中顯示。
  - toBlock()：把內部資料物件再轉換回 &lt;csv-table&gt; 區塊，存回檔案。
  - toPreview()：在編輯器預覽區渲染表格。
    - 內建 parseCSV() 函式，可以正確處理用雙引號包住的欄位（例如 "New York, USA"）。
    - 會將 CSV 轉換成 &lt;table&gt;，並用 &lt;div&gt; 包起來，讓表格支援水平捲動。
4. 編輯介面
  - control()：定義自訂的編輯 UI。
    - 這裡使用一個 &lt;textarea&gt;，讓使用者可以直接輸入或修改 CSV 文字。
    - 當輸入有變動時，會透過 props.onChange() 即時更新資料。

## HTML
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="robots" content="noindex" />
    <link
      href="https://unpkg.com/tabulator-tables@5.5.0/dist/css/tabulator.min.css"
      rel="stylesheet"
    />
    <script src="https://identity.netlify.com/v1/netlify-identity-widget.js"></script>
    <title>Content Manager</title>
  </head>
  <body>
    <script src="https://unpkg.com/decap-cms@^3.0.0/dist/decap-cms.js"></script>
    <script src="https://unpkg.com/tabulator-tables@5.5.0/dist/js/tabulator.min.js"></script>
    <script src="/admin/widgets/csv-test.js"></script>
    <script src="/admin/autoSave.js"></script>
    
    <script>
      CMS.registerPreviewStyle("/admin/preview.css");
    </script>
  </body>
</html>
```

## JavaScript
```javascript
console.log("✅ csv-widget.js loaded");

CMS.registerEditorComponent({
  id: "csv-table",
  label: "CSV Table",
  fields: [{ name: "csv", label: "CSV Content", widget: "text" }],
  pattern: /^<csv-table>([\s\S]*?)<\/csv-table>$/ms,
  fromBlock: function(match) {
    const content = match && match[1] ? match[1].trim() : "";
    return { csv: content || "Name,Age,Note\nAlice,23,\"Likes apples, bananas\"\nBob,30,\"Enjoys running, swimming\"" };
  },
  toBlock: function(data) {
    return `<csv-table>\n${data.csv || "Name,Age,Note\nAlice,23,\"Likes apples, bananas\"\nBob,30,\"Enjoys running, swimming\""}\n</csv-table>`;
  },
  toPreview: function(data) {
    const csvContent = data.csv || "Name,Age,Note\nAlice,23,\"Likes apples, bananas\"\nBob,30,\"Enjoys running, swimming\"";

    // CSV 解析函式，支援雙引號包逗號
    function parseCSV(text) {
      const rows = [];
      const lines = text.split("\n");
      for (let line of lines) {
        const cells = [];
        let match;
        const regex = /"(.*?)"|([^,]+)/g; // 匹配引號內或普通欄位
        while ((match = regex.exec(line))) {
          cells.push(match[1] !== undefined ? match[1] : match[2]);
        }
        rows.push(cells);
      }
      return rows;
    }

    const rows = parseCSV(csvContent);
    const htmlRows = rows.map((r, idx) => {
      if (idx === 0) return "<tr>" + r.map(c => `<th>${c}</th>`).join("") + "</tr>";
      return "<tr>" + r.map(c => `<td>${c}</td>`).join("") + "</tr>";
    });

    // 包一個可水平滑動容器
    return `<div style="overflow-x:auto; width:100%; margin-top:5px;">
      <table border="1" style="border-collapse: collapse; width:100%; text-align:left;">
        ${htmlRows.join("\n")}
      </table>
    </div>`;
  },
  control: function(props) {
    const container = document.createElement("div");
    container.style.display = "flex";
    container.style.flexDirection = "column";
    container.style.width = "100%";

    // 編輯區使用 textarea 直接修改 CSV 文字
    const textarea = document.createElement("textarea");
    textarea.style.flex = "1";
    textarea.style.width = "100%";
    textarea.style.height = "200px";
    textarea.value = props.value || "Name,Age,Note\nAlice,23,\"Likes apples, bananas\"\nBob,30,\"Enjoys running, swimming\"";

    // 文字變動時直接更新
    textarea.addEventListener("input", () => {
      props.onChange(textarea.value);
    });

    container.appendChild(textarea);
    return container;
  }
});
```
