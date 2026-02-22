RESTful API 觀念與實戰指南
============
## 1. 什麼是 API 與 RESTful API？

- **API (應用程式介面)**：簡單來說就是「寫給工程師看的功能使用說明書」。它定義了前端該用什麼格式把請求發給後端，以及後端會回傳什麼格式的資料。
  - 口語：「Call API」指的就是前端對後端的 API 發起一次 HTTP 請求。
- **RESTful API**：REST 是一種軟體設計的「風格」，而 RESTful API 就是指**完美符合 REST 設計風格的 API**。
  - 目的：建立全世界工程師的共通開發默契，降低前後端溝通成本。
   
---
## 2. 成為 RESTful API 的「三大必備條件」

要設計出合格的 RESTful API，必須同時滿足以下三個條件：

### 條件一：使用 HTTP Method 表達資料庫操作 (CRUD)

不再只用 GET 查資料、POST 傳資料，而是將 HTTP Method 與資料庫的四個基本操作緊密結合：
-  `GET`  對應 **R**ead (查詢/讀取數據)
- `POST`  對應 **C**reate (新增數據)
- `PUT`  對應 **U**pdate (修改/更新數據)
- `DELETE`  對應 **D**elete (刪除數據)

### 條件二：使用 URL 路徑表達「資源階層關係」

- URL 上的斜線 `/` 可以翻譯成中文的「的」。
- 以**學生系統**為例：
  - `GET /students`：取得所有學生數據。
  - `GET /students/123`：取得所有學生中**的** id 為 123 的學生數據。
  - `DELETE /students/123`：刪除 id 為 123 的學生數據。
      
### 條件三：Response Body 必須回傳 JSON 或 XML 格式

- 在現代開發中，通常都是回傳 JSON。在 Spring Boot 中，只要加上 `@RestController` 註解並回傳 Java 物件，就能自動達成此條件。

---

## 3. Spring Boot 實戰：RESTful API 專屬註解與實作

在 Spring Boot 中，我們不需要再寫冗長的 `@RequestMapping(value = "/students", method = RequestMethod.POST)`，而是直接使用更簡潔的專屬註解。

### 四大核心註解：
- `@GetMapping`：限制只能用 GET 請求。
- `@PostMapping`：限制只能用 POST 請求。
- `@PutMapping`：限制只能用 PUT 請求。
- `@DeleteMapping`：限制只能用 DELETE 請求。

### 實戰演練：對 `Student` 資源進行 CRUD 實作

以下是在 `StudentController` 中實作四個基本 API 的標準寫法（結合了先前學過的 `@PathVariable` 與 `@RequestBody`）：

```JAVA
@RestController
public class StudentController {

    // 1. Create (新增學生)
    @PostMapping("/students")
    public String create(@RequestBody Student student) {
        return "執行資料庫的 Create 操作";
    }

    // 2. Read (查詢特定學生)
    @GetMapping("/students/{studentId}")
    public String read(@PathVariable Integer studentId) {
        return "執行資料庫的 Read 操作，查詢 ID: " + studentId;
    }

    // 3. Update (更新特定學生資訊)
    @PutMapping("/students/{studentId}")
    public String update(@PathVariable Integer studentId, @RequestBody Student student) {
        return "執行資料庫的 Update 操作，更新 ID: " + studentId;
    }

    // 4. Delete (刪除特定學生)
    @DeleteMapping("/students/{studentId}")
    public String delete(@PathVariable Integer studentId) {
        return "執行資料庫的 Delete 操作，刪除 ID: " + studentId;
    }
}
```

---
## 4. 注意：資安永遠大於設計風格

- **REST 只是一種「建議風格」，不是強制標準。**
- **危險示範**：如果查詢條件包含敏感資訊（如：身分證字號 `B123456789`），按照 REST 風格寫成 `GET /users/B123456789` 會導致個資直接暴露在網址列上，非常危險！
- **正確解法**：遇到敏感資訊時，應果斷放棄 REST 風格，改用 `POST` 方法將資料藏在 Request Body 中傳遞。

