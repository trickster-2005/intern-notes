# Implementation
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

網站編輯流程：
- 使用者在 **Decap CMS** 編輯器中輸入資料  
- 編輯結果自動存入 GitHub 儲存庫  
- **Netlify** 負責自動化部署，將內容交由 **Hexo** 轉換成靜態頁面  
- CSV 與 JSON 經由 Widget 處理，最終以 HTML table 或樹狀結構渲染  
