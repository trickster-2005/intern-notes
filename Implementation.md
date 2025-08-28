# Implementation 實作
## 計畫初期設計
在八月中實作了兩個主要功能：
1. **CSV 編輯器**：基於 Handsontable，提供類似 Excel 的直覺操作。 
    [demo](https://trickster-2005.github.io/intern-final-v0/)
2. **JSON 編輯器**：透過 D3.js 將結構以樹狀圖呈現，方便瀏覽與修改。  
    [demo](https://trickster-2005.github.io/intern-final-v0/tree/)

這兩個功能能夠涵蓋多數結構化資料需求，並且與 Markdown 編輯互補。

## 挑戰與問題

1. **互動功能整合困難**  
   - 在 CMS 編輯環境中，按鈕與互動無法正常載入。  
   - 初步判斷原因可能是 React 渲染生命週期不熟悉，以及 Decap CMS 本身的 sandbox 限制。  

2. **CSV Parsing 的困境**  
   - 不同使用者可能輸入不一致的格式，例如含有逗號的地名或不同的分隔符。  
   - 單純依靠 `split(",")` 難以涵蓋所有情境，因此採用逐字元解析。  
   - 此做法雖然更靈活，但在效能與程式複雜度之間需要權衡。  

3. **資料驗證的兩難**  
   - 嚴格驗證可確保輸出格式正確，但可能降低使用者的自由度。  
   - 寬鬆驗證則能提升易用性，但可能導致不一致的資料輸出。  

## 系統架構

### 整體流程
1.  **使用者編輯**
    - 在 Decap CMS 編輯器中，使用者透過 CSV Widget 的 **textarea** 介面輸入表格資料。
2.  **Widget 轉換**
    - CSV Widget 將輸入的內容包裹在 `<csv-table> ... </csv-table>` 區塊中。
    - 這個區塊直接儲存在 Markdown 檔案內，確保資料與其他內容（文字、圖片）一同版本控製。
    範例：
    `# Example PostHere is my data:<csv-table>place,nameTaiwan,Andy"New York, USA",Amy</csv-table>`
    
3.  **GitHub 儲存**
    - 當使用者點擊「Save」或「Publish」時，Decap CMS 會將修改後的 Markdown 檔案推送到 GitHub repository。
    - 因為 `<csv-table>` 本質上仍是純文字區塊，所以不需要額外的檔案存放格式，直接享有 Git 的版本控管。
        
4.  **Netlify 自動部署**
    - Netlify 偵測到 GitHub 的變更後，觸發 CI/CD pipeline，重新生成靜態網站。
5.  **Hexo 轉換與渲染**
    - Hexo 在編譯時會讀取 Markdown 檔案。
    - 遇到 `<csv-table>` 標籤時，Widget 的 `toPreview()` / `toBlock()` 已經確保格式正確，因此會輸出為 `<table> ... </table>` HTML 結構。
    - 這些 HTML 會被嵌入最終的靜態頁面中。