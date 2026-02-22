Spring MVC
===
## 1. Spring MVC 簡介 
Spring MVC 是 Spring Framework 中的核心 Web 框架，基於 Java Servlet API 建構，採用模型-視圖-控制器 (MVC) 設計模式。它將應用程式劃分為 Model（業務邏輯）、View（界面呈現）和 Controller（請求處理），以輕量級、鬆耦合的方式開發 Web 應用，現已成為主流的後端 API 開發框架。

### 1. 前端與後端的職責差異

- **前端 (Front-end)**：負責網頁的**排版與設計**，主要實作**網頁的外觀**。
- **後端 (Back-end)**：負責數據處理與商業邏輯，主要處理**網頁上動態顯示的資料**。
- **運作模式**：前端向後端詢問所需數據，後端處理後提供給前端。

### 2. 什麼是 Spring MVC？

- 專門用來處理並解決**前端與後端之間的溝通問題**。
- 透過 Spring MVC 的框架，我們可以在 Spring Boot 中建立 API，用來接住前端傳遞過來的請求與參數，並將處理完的資料回傳。

### 3. 其實 Spring MVC 在最一開始創建專案練習時就會用到ㄌ

- 在最初撰寫 Spring Boot 程式並建立 `Controller`（例如在瀏覽器輸入 `http://localhost:8080/test` 來執行特定方法）時，就已經在使用 Spring MVC 的技術。
- **兩大關鍵註解**：
- `@RestController`：定義該類別為處理請求的控制器，並將回傳值轉換為 JSON 等格式。
- `@RequestMapping`：負責將特定的 URL 路徑對應到指定的 Java 方法。

#### 然而在正式深入 Spring MVC 的各項 API 開發前，還得先了解前後端溝通的最基礎規範：**HTTP 協議 (HTTP Protocol)！**
