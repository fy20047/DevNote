Lombok 用法
===

### 1. Lombok 特色

在 Spring Boot 專案中，使用 Lombok 能夠大幅縮減重複的程式碼，但在 JPA Entity 層使用`@Data`可能會與 Hibernate/JPA 的代理機制 (Proxy Mechanism) 產生衝突。

---

### 2. 專案實作

應用於所有與資料庫對應的 Entity Class：

- `backend/.../entity/Admin.java` (涉及敏感欄位 password)
- `backend/.../entity/Order.java` (涉及關聯欄位與預設值)
- `backend/.../entity/RefreshToken.java` (涉及關聯與 Token)
- `backend/.../entity/Tire.java` (涉及庫存預設值)


這邊不用`@Data`，改拆解為以下組合，確保對 Entity 的行為能夠控制：

```Java
@Entity
@Table(name = "admins")
@Getter                 // 1. 產生 getter
@Setter                 // 2. 產生 setter
@Builder                // 3. 實作 Builder Pattern，提升物件建立的可讀性
@NoArgsConstructor      // 4. JPA 規範：必須有無參數建構子供 Reflection 使用
@AllArgsConstructor     // 5. Builder 模式所需：Builder 需要全參數建構子來初始化物件
public class Admin {
    // ...
}
```

**這邊學到一個蠻酷的技巧：`@Builder.Default`**

在 `Order` 與 `Tire` 等實體中，我有設定欄位ㄉ初始值 (如訂單狀態、庫存)。

-   **問題**：Lombok 的 `@Builder` 預設會忽略欄位宣告時的賦值，導致 `new` 出來的物件該欄位為 `null` 或 `0`。
-   **解法**：強制加上 `@Builder.Default`。
```Java
@Builder.Default
@Column(nullable = false)
@Enumerated(EnumType.STRING)
private OrderStatus status = OrderStatus.PENDING; // 確保 Builder 建立時預設為 PENDING
```
---

### 3. 為什麼會有人不建議 Entity 用 `@Data`？

`@Data` 是一個複合註解，包含了 `@ToString`, `@EqualsAndHashCode`, `@Getter`, `@Setter`, `@RequiredArgsConstructor`，雖然方便，但在 ORM 中亂用會出大事。

#### 風險一：無窮遞迴 (StackOverflowError)

- **原因**：`@Data` 產生的 `toString()` 會印出所有欄位。若 Entity 之間存在雙向關聯 (Bi-directional Relationship，例如 Order <-> User)，`Order.toString()` 會呼叫 `User.toString()`，`User` 又呼叫 `Order`，就會導致無窮迴圈。  
- **後果**：應用程式崩潰，拋出 `StackOverflowError`。

#### 風險二：效能殺手 (Lazy Loading Trigger)

-   **原因**：`@EqualsAndHashCode` 與 `@ToString` 會存取物件的所有欄位。
- **後果**：當我們在 Log 中印出 Entity，或將 Entity 放入 `Set/Map` 時，這些方法會強制 JPA 去資料庫撈取設定為 `Lazy Loading` (延遲加載) 的關聯欄位，導致 **N+1 Query** 問題。
- 若 Session 已關閉，甚至會拋出 `LazyInitializationException`。 

#### 風險三：集合操作異常 (Broken Set Contract)

-   **原因**：JPA Entity 的 `equals()` 與 `hashCode()` 應該基於 **Business Key** 或 **固定不變的 ID**。但 `@Data` 預設會使用所有欄位計算 Hash。 
-  **情境**：
    1. 建立一個新 Entity (此時 ID 為 null)。
    2. 放入 `HashSet` (基於 null 計算 Hash 位置)。
    3. 儲存到資料庫 (ID 被賦值，欄位改變 -> HashCode 改變)。
    4. 試圖從 `HashSet` 中移除該物件 -> **失敗**，因為 HashCode 變了，Map/Set 找不到原本的位置。 
- **後果**：導致記憶體洩漏 (Memory Leak) 或邏輯錯誤。

#### 風險四：資安隱憂 (Security Leak)

- **原因**：`@ToString` 會無差別輸出所有欄位。
- **實際案例**：在 `Admin.java` 中包含 `password` 欄位。若使用 `@Data`，當系統發生錯誤並記錄 Log 時，加密後的密碼 Hash (甚至明碼) 就會直接暴露在 Log 檔案中。

---

### 4. 總結

在專案 (Tire Ordering System) 中，針對 Entity 的使用如下：

1.  **安全性優先**：捨棄 `@Data`，改用明確的 `@Getter/@Setter` 組合，防止敏感資訊外洩與不可預期的行為。
2.  **相容性**：保留 `@NoArgsConstructor` 以符合 JPA 規範 (Hibernate 使用 Reflection 實例化)。
3.  **可讀性**：引入 `@Builder` 模式，解決 Entity 欄位過多時，建構子參數難以辨識的問題，並搭配 `@Builder.Default` 確保業務邏輯的預設狀態正確。

