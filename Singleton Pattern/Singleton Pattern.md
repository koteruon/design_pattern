# Singleton Pattern (單例模式)

> **單例模式**是一種創建型設計模式，可以確保一個類別只有一個實例，並且提供一個全域接觸點。

### 單例模式結構

![單例模式結構](./%E5%96%AE%E4%BE%8B%E6%9E%B6%E6%A7%8B.png)

### 單例模式的應用場景

1. **如果程式碼的某個類對於所有客戶端只有一個可用的實例**
    > 單例模式禁止偷過特殊建構方法以外的任何方式來創建自身類的實例
2. **如果你需要加嚴格地控制全域變數**
    > 單例他能保證類只存在一個實例

### 優缺點

:o:**優點**

1. 保證一個類只有一個實例
2. 獲得一個指向實例地全域接觸點
3. 僅在首次請求單例對象的時候才進行初始化

:x:**缺點**

1. 違反單一職責原則。同時解決兩個問題
2. 可能掩蓋不良設計，比如程式碼各組件之間相互了解過多等
3. 在多執行緒環境下需要進行特殊處理，避免多執行緒多次創建單例對象
4. 單例的客戶端程式碼單元測試可能比較困難，因為需多測試框架以基於繼承的方式創建對象。但單例類的建構式是私有的，而且絕大部分語言無法重寫靜態方法，需要想出模擬單例的方法。

## 單例模式(Singleton Pattern)

### 為什麼要用單例模式？

-   **只需要一個實例地例子**
    1. 執行緒池
    2. 快取
    3. 對話方塊
    4. 處理偏好設定
    5. 註冊設定的物件
    6. 做紀錄(logging)的物件
    7. 顯示卡驅動
-   **當實例不只一個的時候，會遇到哪些問題？**
    1. 不正確的程式行為
    2. 過度使用資源
    3. 產生錯誤的結果
-   **與全域變數有什麼差別**
    1. 全域變數會在程式開始執行期的時候建立，如果物件占用很多資源，卻從不使用很浪費。單例只在真正需要的時候才建立物件
    2. 使用全域變數可能會造成namespace的混亂

### 經典的單例模式(無多執行緒)

```java
// NOTE: This is not thread safe!

public class Singleton {
    private static Singleton uniqueInstance; // 我們用一個static變數來保存Singleton

    private Singleton() {} // 建構式宣告成private，只有Singleton可以實例化這個類別

    public static Singleton getInstance() { // 提供一個管道來讓你實例化這個類別
        if (uniqueInstance == null) { // 還沒有建立過這個實例
            uniqueInstance = new Singleton(); // 運用私有的建構式將他實例化，如果用不到就不會被建立，這是一種惰性(lazy)實例化
        }
        return uniqueInstance; // 回傳他的實例
    }

    // other useful methods here
    public String getDescription() {
        return "I'm a classic Singleton!";
    }
}
```

> [!TIP]
>
> 單例模式可以確保一個類別只有一個實例，並且提供一個全域接觸點

### 遇到的問題與想法

> [!WARNING]
>
> 多執行緒中，可能一次創建了兩個實例出來

![單例模式執行緒問題](./%E5%96%AE%E4%BE%8B%E6%A8%A1%E5%BC%8F%E5%9F%B7%E8%A1%8C%E7%B7%92%E5%95%8F%E9%A1%8C.png)

### 雙重檢查鎖版本(多執行緒)

