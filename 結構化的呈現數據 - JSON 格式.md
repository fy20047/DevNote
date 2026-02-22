結構化的呈現數據 - JSON 格式
===============================
在開發廣益輪胎行的訂單系統時，前後端傳遞如「輪胎庫存狀態」、「訂單詳細資訊」等資料，都是依賴 JSON 格式來溝通。

### 1. 為什麼需要 JSON？

- **痛點**：過去如果回傳普通的字串（例如："第一個是蘋果、第二個是橘子"），這對我們來說很好懂，但對電腦與前端程式來說難以解析與應用。
- **解決方案**：JSON (JavaScript Object Notation) 是一種輕量級的資料交換格式。目的是用**更簡單、更直覺且結構化**的方式來呈現數據，讓前後端能極高效率地讀取、傳遞資料，這個觀念對於串接 RESTful API 非常關鍵！

### 2. JSON 的基本語法與結構

- **物件 (Object)**：使用一對大括號 `{}` 來表示一個物件。
- **鍵值對 (Key-Value Pair)**：
  - 資料在 JSON 中是以 `Key: Value` 的形式存在。
  - **Key**：就像是 Java 中的變數名稱，**一定要用雙引號 `""` 包起來**。 
  - **Value**：則是該變數對應的值。
- **多筆資料**：使用逗號 `,` 來分隔不同的 Key-Value 配對。
- **範例**：
    ```JSON
    {
      "id": 123,
      "name": "Judy"
    }
    ```

### 3. JSON 支援的 Value 資料類型

JSON 的 Value 可以放入多種常見的資料型態：

- **字串 (String)**：如 `"Hello"`（需用雙引號）
- **整數/浮點數 (Number)**：如 `123` 或 `1.111`
- **布林值 (Boolean)**：如 `true` 或 `false`
- **陣列/列表 (List/Array)**：
  - 使用中括號 `[]` 來表示。可以用來存放多個相同類型的值。
  - 範例：`"appleList": ["apple1", "apple2", "apple3"]`

### 4. 改寫純文字回傳值

將原本難以被程式解析的口語字串：`"第一個是蘋果、第二個是橘子"`
改寫成結構化的 JSON 格式，前端收到後就能輕鬆轉化為陣列來渲染畫面：
```JSON
{
  "productList": ["蘋果", "橘子"]
}
```

這篇文章把前面學到的 `@RequestMapping` 與 JSON 格式結合在一起，是非常具有實戰意義的一集。在實際開發全端系統（例如你正在開發的廣益輪胎行系統）時，如何把後端的庫存、訂單資料轉換成前端看得懂的格式回傳，用的就是這篇的技巧！

JSON 格式 - `@RestController`
====

### 1. Spring Boot 回傳 JSON 的「兩大步驟」

要讓 Spring Boot 方法自動回傳 JSON 格式，只需要完成以下兩個步驟：
- **步驟一：在 Class 上面加上 `@RestController` 註解**
  - `@RestController` 是一個多功能的強大註解，它不僅能讓 Class 變成 Spring 容器中的 Bean、讓裡面的 `@RequestMapping` 生效，最重要的功能是它能自動將方法的返回值轉換成 JSON 格式。
- **步驟二：將方法的返回值改成「Java 物件 (Object)」**
  - 不直接回傳像是 `String` 之類的格式，而是先建立一個 Java Class（例如 `Store` 或 `Order`），在裡面定義好變數（如 `List<String> productList`）並寫好 `getter` 和 `setter`。
  - 在 `@RequestMapping` 對應的方法中，`new` 出這個物件，把資料塞進去後直接 `return` 這個物件。

### 2. Spring Boot 的底層自動轉換魔術

當我們完成上述兩步後，Spring Boot 會在背景做一件事：當方法 `return` 一個 Java 物件時，Spring Boot 會自動去讀取這個物件裡面的變數，**把變數名稱變成 JSON 的 `Key`，變數裡面的值變成 JSON 的 `Value`**。

例如回傳包含 `productList` 變數的 `Store` 物件，前端收到的 HTTP Response Body 就會自動變成：
```JSON
{
  "productList": [
    "蘋果",
    "橘子"
  ]
}
```

### 3. `@RestController` vs `@Controller`

這兩個註解代表了網頁開發的歷史演進：

-   **`@Controller`（早期作法）**：早期沒有前後端分離時使用。它的作用是將返回值轉換為「前端模板的名字」（例如尋找一個叫 index.html 或 index.jsp 的檔案）。現在只有在使用 Thymeleaf 或 JSP 這類模板引擎時才會用到。
-   **`@RestController`（現代主流）**：隨著前後端分離成為主流，前後端統一用 JSON 溝通。`@RestController` 其實就是 `@Controller` 加上 `@ResponseBody` 的組合體，專門用來把返回值轉成 JSON 數據。**網路上爬文看起來現在開發 API 大多都是優先使用 `@RestController`。**

補充：JSON 與 YAML 的差異比較
=====

雖然 JSON 和 YAML 都是用來結構化呈現資料的語言，但它們的設計哲學和實務應用場景有很大的不同：

### 1. 語法結構與可讀性

- **JSON**：依賴大量的符號，例如大括號 `{}` 代表物件、中括號 `[]` 代表陣列，且變數名稱（Key）與字串必須嚴格使用雙引號 `""` 包裹。格式要求非常嚴謹，少一個逗號或引號就會報錯，讓電腦解析起來可以快且準。
- **YAML**：捨棄了繁瑣的括號，主打人類ㄉ可讀性。它利用**空白鍵縮排**來表示資料的層級關係。視覺上看起來非常乾淨、直覺，就像在列大綱一樣。

### 2. 註解支援 (Comments)

- **JSON**：標準規範中**不支援**任何註解。
- **YAML**：支援 `#` 字號進行單行註解。

### 3. 實務上的應用場景
- **JSON**：**是前後端資料傳輸的絕對標準**。在開發全端網站的 API 時，前端發出的 HTTP Request 與後端回傳的 Response，絕大多數都是採用 JSON 格式來交換數據。
- **YAML**：**系統與環境設定檔的首選**。在 Spring Boot 開發中，似乎通常都會將預設的 `application.properties` 改寫為階層更清晰的 `application.yaml`，我的輪胎專案也是這樣改。此外，在建置 CI/CD 自動化流程、編寫 Docker Compose 腳本或 K8s 部署檔等 DevOps 相關技術時，YAML 也幾乎是最核心的共通語言。

> 總結觀念：「寫 API 傳資料用 JSON，寫 Spring Boot 專案設定或 CI/CD 部署流程用 YAML」。



