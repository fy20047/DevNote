HTTP 協議介紹 & 溝通規範
========================

### 1. 什麼是 HTTP 協議 (HTTP Protocol)？

- **概念**：HTTP 就像是前後端溝通的文法或合約，負責規定資料的**傳輸格式**，確保前端與後端能聽懂彼此的話，有效進行資料交換。
- **運作模式**：一次完整的 HTTP 溝通由「**一個 HTTP Request (請求) + 一個 HTTP Response (回應)**」組成（前端發問，後端回答）。

### 2. HTTP Request (前端發出的請求) 

當前端向後端要資料時，請求中會包含四個主要部分：
1. **HTTP Method (請求方法)**：表達這次請求的「動作目的」（常見如 GET、POST、PUT、DELETE）。
2. **URL (網址)**：請求的目標路徑，對應到要尋找的資源（例如：`http://localhost:8080/test`）。
3. **Request Header (請求標頭)**：放置請求時的通用資訊或設定檔（比較進階的設定）。
4. **Request Body (請求本體)**：用來傳遞請求參數給後端。只有特定的 Method（如 POST、PUT）才能攜帶 Request body。

### 3. HTTP Response (後端回傳的結果) 

當後端處理完請求並回傳給前端時，回應中包含三個主要部分：
1. **HTTP Status Code (狀態碼)**：快速告訴前端這次請求是「成功」還是「失敗」，這裡可以分成五大類：
    ```
    1xx : 資訊
    2xx : 成功
    3xx : 重新導向
    4xx : 前端請求錯誤
    5xx : 後端處理有問題
    ```
2. **Response Header (回應標頭)**：放置後端回傳的通用資訊。
3. **Response Body (回應本體)**：**這是最重要的部分**，裡面裝著後端真正要給前端的數據資料（例如：商品列表或查詢結果），而前端則會將這裡的數據拿去和排版結合。

### 4. 測試工具的概念 (如 API Tester, Postman)

- 在開發 Spring Boot 後端時，我們通常會使用工具（像是 Talend API Tester、Postman）來模擬前端發出 HTTP Request。
- 這些工具的底層邏輯都大同小異，嚴格遵守 HTTP 協議的規範來設定 Method、URL 和 Body，藉此驗證我們的後端程式是否能正確回傳 HTTP Response。

## URL 路徑對應與 `@RequestMapping`

### 1. URL 網址的格式拆解

在發起 HTTP Request 時，網址 (URL) 有著嚴格的格式規範。這邊以 `http://localhost:8080/test` 為例：
- **使用的協議**：`http://`（通常是 HTTP 或 HTTPS）。
- **域名 (Domain)**：`localhost`（本機端測試用），實務上會是像 `youtube.com`。
- **Port (端口)**：`:8080`（選填），作為後端伺服器預設的對外窗口。
- **URL 路徑 (Path)**：`/test`，從域名或 Port 後面的第一個斜線 `/` 開始，後面所有的字串都是 URL 路徑，它會決定這個請求要交給哪份程式處理。

### 2. `@RequestMapping` 的用途

- **功能**：用來將前端請求的 URL 路徑對應到 Spring Boot 裡特定的 Java 方法上。
- **寫法範例**：
    ```Java
    @RequestMapping("/product")
    public String product() {
        return "第一個是蘋果、第二個是橘子";
    }
    ```
    當前端發出 URL 路徑為 `/product` 的請求時，Spring Boot 就會自動去執行這段 `product()` 方法，並將回傳值放入 Response Body 交給前端。
    

### 3. 使用 `@RequestMapping` 的前提
- **一定要搭配 Controller 註解**：只有在類別 (Class) 的最上方加上 `@Controller` 或 `@RestController`，裡面的 `@RequestMapping` 才會生效。如果忘記加那 Spring Boot 就會完全忽略掉這些路徑設定。

### 4. 注意事項
- 一個 Controller 裡面可以寫無限多個 `@RequestMapping` 來對應不同的功能（例如 `/products`、`/orders`），但隨著專案變大，要小心不要定義到重複的 URL 路徑。也就是說每個路徑對應的方法必須是唯一的，否則程式在啟動時會發生衝突而報錯。

