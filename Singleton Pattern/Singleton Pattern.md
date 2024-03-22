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
4. 漸少開銷，因為只有一個實例，減少記憶體開銷，又不需要頻繁實例化，減少性能開銷
5. 避免某個資源多重占用，例如文件執行寫操作時，單例可以保證對文件只有一個寫操作

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
    2. 使用全域變數可能會造成 namespace 的混亂
-   **使用場景**
    1. 某個類的實例需要被頻繁的創建，試用後需要頻繁銷毀
    2. 某個類的實例化耗時、占用資源多且使用頻繁
    3. 某個類的實例頻繁的與數據庫或文件交互
    4. 某個類包含全域狀態，或在業務邏輯上僅應該有單一實例

### 經典的單例模式(執行緒不安全)

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

### 遇到的問題

> [!WARNING]
>
> 多執行緒中，可能一次創建了兩個實例出來

![單例模式執行緒問題](./%E5%96%AE%E4%BE%8B%E6%A8%A1%E5%BC%8F%E5%9F%B7%E8%A1%8C%E7%B7%92%E5%95%8F%E9%A1%8C.png)

**使用 synchronized 來處理多執行緒的問題**

```java
public class Singleton {
    private static Singleton uniqueInstance;

    private Singleton() {}

    public static synchronized Singleton getInstance() { // 增加synchronized關鍵字，讓每一個執行緒都必須等到輪到他時，才可以進入這個方法
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }

    // other useful methods here
    public String getDescription() {
        return "I'm a thread safe Singleton!";
    }
}
```

> [!WARNING]
>
> **同步的代價很高，而且同步只有第一次在執行這個方法時派上用場**，所以當你將 uniqueInstance 設為單例之後，同步化就不需要了。

### 多執行緒可能的方法

1. 如果 getInstance()的性能對你的應用程式來說沒那麼重要，那就放著不管
2. 使用急性(eager)建立實例，而不是惰性(lazy)建立。

```java
public class Singleton {
    private static Singleton uniqueInstance = new Singleton(); // 用靜態初始設定式來建立單例實例，這是執行緒安全

    private Singleton() {}

    public static Singleton getInstance() {
        return uniqueInstance; // 因為已經建立了，直接回傳實例
    }

    // other useful methods here
    public String getDescription() {
        return "I'm a statically initialized Singleton!";
    }
}
```

3. **靜態內部類**。外部類被加載時，靜態內部類不會跟著一起被加載，而是在靜態內部類被使用的時候才會被加載並實例化，所以靜態內部類具有 Lazy 加載的優點，同時保證線程安全

```java
public class Singleton {
    private Singleton() {}

    public static Singleton getInstance() {
        return SingletonInstance.uniqueInstance; // 只有呼叫的時候才會加載靜態內部類
    }

    private static class SingletonInstance { // 靜態內部類
        private static Singleton uniqueInstance = new Singleton();
    }
}
```

### 雙重檢查鎖版本(DCL)(多執行緒)

**使用雙重檢查鎖，在 getInstance()中減少同步化**

-   volatile 保證可見性，避免 JVM 指令重排引發的問題

```java
public class Singleton {
    private volatile static Singleton uniqueInstance; // volatile保證可見性，避免JVM指令重排引發的問題

    private Singleton() {}

    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            synchronized (Singleton.class) { // 只有在第一次時同步，並鎖上Class
                if (uniqueInstance == null) { // 進入這個區塊後，還要再次檢查，如果他還是null，就建立一個實例
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

-   解決反序列化破壞單例模式

```java
public class Singleton implements Serializable { // 序列化

    private volatile static Singleton uniqueInstance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }

    private Object readResolve() { // 加上readResolve解決反序列化問題
        return uniqueInstance;
    }
}
```

-   解決反射破壞單例模式

```java
public class Singleton {
    private volatile static Singleton uniqueInstance;

    private Singleton() {
        if(uniqueInstance != null) {
            throw new RuntimeException(); //使用反射創建時例時會拋出異常
        }
    }

    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

### 可以繼承的版本

```java
public class Singleton {
    protected static Singleton uniqueInstance;

    // other useful instance variables here

    protected Singleton() {}

    public static synchronized Singleton getInstance() {
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }

    // other useful methods here
}

// ----------------------------------------------------------------

public class HotterSingleton extends Singleton {
    // useful instance variables here

    private HotterSingleton() {
        super();
    }

    // useful methods here
}
```

## 使用 enum 來處理單例(最佳方法)

對同步的擔憂、類別載入的問題、反射、序列化/反序列化問題，都可以使用 enum 來建立單例並解決，**JVM 會保證 ENUM 類不能被反射且構造器只能被調用一次**

```java
public enum Singleton {
    UNIQUE_INSTANCE;

    // other useful fields here

    // other useful methods here
    public String getDescription() {
        return "I'm a thread safe Singleton!";
    }
}

// ----------------------------------------------------------------

public class SingletonClient {
    public static void main(String[] args) {
        Singleton singleton = Singleton.UNIQUE_INSTANCE;
        System.out.println(singleton.getDescription());
    }
}
```

### 沒有蠢問題

**Q: 難道不能建立一個類別，將裡面的所有方法和變數都定義成 Static 嗎？**

A: 由於 java 處理靜態初始化的方式，這種作法可能導致混亂，尤其有許多類別牽涉其中。這樣做會導致許多不易察覺的 bug，**這些 bug 與初始化順序有關**。

**Q: 使用兩個類別載入器(Classloader)有可能各自建立他們自己的單例？**

A: 的確，每一個類別載入器都定義了自己的名稱空間。如果有兩個以上的類別載入器，就有可能多次載入同一個類別，因為類別的版本不只一個，所以單例的實例也不只一個。處理這個問題的方法之一是**自己指定類別載入器**。

**Q: 使用反射(reflection)與序列化(Serialization) / 反序列化(Deserialization)呢？**

A: 反射和序列化/反序列化會讓單例出問題，使用 ENUM 可以解決。

**Q: 單例有沒有違反鬆耦合原則？在程式碼中，每一個依賴單例的物件都會與那一個物件緊密地耦合**

A: 確實，這是單例模式為人詬病的一點。單例很容易違反這一條原則：**當你修改單例時，很有可能也要修改與它有關的每一個物件**。

**Q: 單例有沒有違反單一職責原則(Single Pesponsibility Principle)？**

A: 單例違反單一職責原則，因為他不但**負責管理他的唯一實例**，並同時**提供全域性的接觸點**。但這個類別用一種機制來管理它自己的實例，會讓整體的設計更簡單。

**Q: 可以用單例來做子累嗎？**

A: 用單例來做子類別的問題在於**建構式是 private 的**，無法用 privat 的建構式來繼承類別，如果改成 protected，它就不是單例了。修改建構式還有一個問題，單例是**以 static 變數為基礎**來建構了，所以直接繼承，它的衍生類別都會**共用同一個實例變數**，你應該不想這樣，所以，為了繼承，必須在基底類**實作註冊儲存體(registry)**。所以單例不一定適合放入程式庫的解決方案，如果應用程式裡有大量的單例，就應改檢討，**單例不是讓你大量使用的**。

**Q: 全域變數為何不如單例？**

A: 惰性實例化 vs 急性實例化。全域變數沒有確保類別只有一個實例，只有確保提供全域性的接處。而且全域變數往往會引誘開發者使用許多小物件的全域參考，**汙染名稱空間**。
