# Factory Method Pattern (工廠模式)

> **工廠模式**是一種創建型設計模式，在父類提供一個創建對象的方法，允許子類決定實例化對象的類型

### 工廠模式結構

![工廠模式結構](./%E5%B7%A5%E5%BB%A0%E6%A8%A1%E5%BC%8F%E6%9E%B6%E6%A7%8B.png)

### 工廠模式的應用場景

1. **在寫程式時，如果無法預知對象確切類別以及他的依賴關係時**
    > 工廠方法會將創建產品的程式碼和實際使用產品的程式碼分離，能在不影響其他程式碼的情況下擴展產品創建部分程式碼
2. **如果希望用戶能夠擴展你的 library 或 framework 的內部組件**
    > 將框架中構造組件的程式碼集中到一個工廠方法中，並在繼承組件之外允許任何人對該方法進行重寫
3. **如果希望復用現有對象來節省系統資源，而不是每次都重新創建對象**
    1. 首先，需要創建儲存空間來存放所有已創建的對象
    2. 當其他人請求一個對象時，程式在對象池中搜尋可以用的對象
    3. 然後將他返回給客戶端程式
    4. 如果沒有可用的對象，程式則創建一個新對象(將他添加到對向池中)
        > 因此，需要有一個既能夠創建新對象，又可以復用現有對象的方法，和工廠方法非常相像

### 優缺點

:o:**優點**

1. 可以避免創建者和具體產品之間的緊密耦合
2. 單一職責原則。可以將產品創建程式碼放在程式的單一個位置，使得程式碼更容易維護
3. 開放封閉原則。無須更改現有客戶程式碼，就可以在程式中引入新的產品類型

:x:**缺點**

1. 工廠模式需要引入許多新的子類，可能讓程式碼變得複雜。最好的情況是將該模式引入創建者類的現有層次結構中(產品類太多時，為每個產品創建子類並無太大必要，可以在子類中復用基類中的控制參數)

## 簡單工廠(Simple Factory)

### 原始的模型架構

1. 首先你有一個 pizza，都會經過 prepare、bake、cut、box

```java
public class PizzaStore {

    public Pizza orderPizza() {
        Pizza pizza = new Pizza(); // 為了更有彈性，我們想把它寫成抽象類別或介面，遺憾的是，抽象類別或介面都無法直接實例化。

        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();

        return pizza;
    }
}
```

2. 增加更多種類的 pizza，透過傳入參數決定

```java
public Pizza orderPizza(String type) {
    Pizza pizza;

    /* 根據pizza的種類來實例化正確的具體類別，並指派給pizza實例變數 */
    if (type.equals("cheese")) {
        pizza = new CheesePizza();
    } else if (type.equals("greek")) {
        pizza = new GreekPizza();
    } else if (type.equals("pepperoni")) {
        pizza = new PepperoniPizza();
    }

    /* 做一些準備工作(揉麵團、放上醬料和佐料)，烘烤、切開、放到盒子 */
    /* 每一種pizza的子型態(CheesePizza、GreekPizza...)都知道要怎麼準備他 */
    pizza.prepare();
    pizza.bake();
    pizza.cut();
    pizza.box();

    return pizza;
}
```

### 遇到的需求與問題

3. 需要增加 ClamPizza 和 VeggiePizza，並刪除 GreekPizza

```java
public Pizza orderPizza(String type) {
    Pizza pizza;

    /* 這是會變得部分，會隨著pizza品項不斷改變而反覆修改這段程式 */
    if (type.equals("cheese")) {
        pizza = new CheesePizza();
    } else if (type.equals("greek")) { // 要刪除的greekpizza
        pizza = new GreekPizza();
    } else if (type.equals("pepperoni")) {
        pizza = new PepperoniPizza();
    } else if (type.equals("clam")) { // 新增clampizza
        pizza = new ClamPizza();
    } else if (type.equals("veggie")) { // 新增veggiepizza
        pizza = new VeggiePizza();
    }

    /* 這是我們認為不變的部分，因為多年來，準備、烘烤、包裝都沒有變過。 */
    pizza.prepare();
    pizza.bake();
    pizza.cut();
    pizza.box();

    return pizza;
}
```

> [!NOTE]
>
> 上面中間的程式**並未**拒絕修改。每次 pizza 改變品項，我們就必須修改這段程式碼。

### 建構簡單工廠(封裝物件的建立)

4. 將會變得部分移出 oderpizza()，移到另一個單純負責建立 pizza 的物件裡面(SimplePizzaFactory)

```java
public class SimplePizzaFactory {

    public Pizza createPizza(String type) {
        Pizza pizza = null;

        if (type.equals("cheese")) {
            pizza = new CheesePizza();
        } else if (type.equals("pepperoni")) {
            pizza = new PepperoniPizza();
        } else if (type.equals("clam")) {
            pizza = new ClamPizza();
        } else if (type.equals("veggie")) {
            pizza = new VeggiePizza();
        }
        return pizza;
    }
}

// ----------------------------------------------------------------

public class PizzaStore {
    SimplePizzaFactory factory;

    public PizzaStore(SimplePizzaFactory factory) { // 在建構是裡接收傳來的工廠
        this.factory = factory;
    }

    public Pizza orderPizza(String type) {
        Pizza pizza;

        pizza = factory.createPizza(type); // 將訂單的種類傳給工廠，用他來建立pizza，這裡也沒有具體實例化了！

        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();

        return pizza;
    }

}
```

### 沒有蠢問題

**Q: 這樣做有什麼好處？只是把問題搬到另一個物件裡面**

A: SimplePizzaFactory 可能有許多用戶端，未來可能還有 PizzaShopMenu 類別會使用這個工廠來取得 pizza，用來顯示 pizza 的說明和價格。也可能還有 HomeDelivery 類別會使用這個工廠來取得 pizza，採取和 PizzaStore 不一樣的方式來處理 pizza。**所以將建立 pizza 的程式封裝成一個類別，在處理變動時，只需要修改一個地方。**

**Q: 有些會將工廠定義成靜態(static)方法，有什麼不同？**

A: 將簡單工廠定義成靜態方法很常見，稱作靜態工廠，如此一來就不需要為了使用 create(建立)方法而進行工廠物件實例化，**但這個做法有一項缺點，你無法繼承並修改 create 方法的行為。**

### 定義簡單工廠

**簡單工廠不是設計模式，比較像習慣寫法**

![簡單工廠例子](./%E7%B0%A1%E5%96%AE%E5%B7%A5%E5%BB%A0%E4%BE%8B%E5%AD%90.png)

> [!NOTE]
>
> _實作介面_ **不一定**代表 _在類別宣告式裡面使用 implements 來編寫一個實作了 Java 介面的類別_ 。廣義來說，讓一個具體的類別實作超型態(可能是抽象類別或是介面)的方法，仍然可以視為 _實作_ 那個超型態的 _介面_。

## 工廠方法(Factory Method)

5. 希望加盟 Pizza 店，因此創建兩個 NYPizzaFactory 和 ChicagoPizzaFactory，製作不同風味的 pizza。

```java
NYPizzaFactory nyFactory = new NYPizzaFactory();
PizzaStore nyStore = new PizzaStore(nyFactory);
nyStore.orderPizza("Veggie");

// ----------------------------------------------------

ChicagoPizzaFactory chicagoFactory = new ChicagoPizzaFactory();
PizzaStore chicagoStore = new PizzaStore(chicagoFactory);
chicagoStore.orderPizza("Veggie");
```

> [!WARNING]
>
> 加盟的 nyStore 和 chicagoStore 開始採取自創的流程，因此我們需要把變得部分移出，利用抽象物件保留不變的部分(Pizza 店和 Pizza 的製作過程綁再一起)，同時保持一些彈性由子類自己決定

### 為 pizza 店設計框架、讓子類做決定

6. 在 PizzaStore 類別裡，將所有的 pizza 製作動作局部化(localize)，同時讓連鎖店可以自由地製作地區風味

```java
public abstract class PizzaStore { // 宣告成抽象類別，讓子類實作工廠方法

    public Pizza orderPizza(String type) {// 想要的話，可以將其設定為final，強迫子類一定要使用他
        Pizza pizza;

        pizza = createPizza(type); // 透過抽象方法，取得pizza，不再是factory物件的方法

        /* 保留不變的地方，讓子類都能保持一致 */
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }

    protected abstract Pizza createPizza(String item); // 現在「工廠方法」是在PizzaStore裡面的抽象方法，也強迫所有子類必須實現這個方法
}
```

7. 創建兩間 Pizza 店，NYPizzaStore 和 ChicagoStylePizzaStore，他們都繼承 PizzaStore，子類別會用自己的 createPizza 方法製作地區風味 pizza

```java
public class NYPizzaStore extends PizzaStore {// 繼承PizzaStore，所以繼承orderPizza()和其他方法

    /* 必須實作createPizza，因為他是抽象的，並且會回傳一個Pizza，子類別全權負責決定他要實例化哪一種具體的Pizza */
    Pizza createPizza(String item) { // 為每一種pizza製作紐約風味
        if (item.equals("cheese")) {
            return new NYStyleCheesePizza();
        } else if (item.equals("veggie")) {
            return new NYStyleVeggiePizza();
        } else if (item.equals("clam")) {
            return new NYStyleClamPizza();
        } else if (item.equals("pepperoni")) {
            return new NYStylePepperoniPizza();
        } else return null;
    }
}

// ----------------------------------------------------------------

public class ChicagoPizzaStore extends PizzaStore {

    Pizza createPizza(String item) { // 為每一種pizza製作芝加哥風味
        if (item.equals("cheese")) {
            return new ChicagoStyleCheesePizza();
        } else if (item.equals("veggie")) {
            return new ChicagoStyleVeggiePizza();
        } else if (item.equals("clam")) {
            return new ChicagoStyleClamPizza();
        } else if (item.equals("pepperoni")) {
            return new ChicagoStylePepperoniPizza();
        } else return null;
    }
}
```

**Q: 子類如何做決定？**

A: 從 PizzaStore 的 orderPizza 想，在方法裡面用 Pizza 物件做了很多事，但 Pizza 是抽象的，**具體型態只會在子類別裡面建立**，換句話說，他們是**解耦的**。所以要看你向哪一個 pizza 店訂購 Pizza，是 NYPizzaStore 或 ChicagoPizzaStore。所以子類別做出及時的決定嗎？沒有，但從 orderPizza 的角度，決定做哪一種 Pizza 的就是那個子類別。所以子類別其實沒有「做決定」，決定向哪一間訂餐的人是你，但是他們確實決定了做出來的是哪種 Pizza。

8. 製作 Pizza 介面(產品)和各式風味的 Pizza(具體產品)

```java
public abstract class Pizza {
    String name; // 名稱
    String dough; // 餅皮
    String sauce; // 醬料
    ArrayList<String> toppings = new ArrayList<String>(); // 一組配料

    void prepare() { // 準備流程是一組特定順序的步驟
        System.out.println("Prepare " + name);
        System.out.println("Tossing dough...");
        System.out.println("Adding sauce...");
        System.out.println("Adding toppings: ");
        for (String topping : toppings) {
            System.out.println("   " + topping);
        }
    }

    /* 烘烤、切開、包裝的基本預設值 */
    void bake() {
        System.out.println("Bake for 25 minutes at 350");
    }

    void cut() {
        System.out.println("Cut the pizza into diagonal slices");
    }

    void box() {
        System.out.println("Place pizza in official PizzaStore box");
    }

    public String getName() {
        return name;
    }

    public String toString() {
        StringBuffer display = new StringBuffer();
        display.append("---- " + name + " ----\n");
        display.append(dough + "\n");
        display.append(sauce + "\n");
        for (String topping : toppings) {
            display.append(topping + "\n");
        }
        return display.toString();
    }
}

// ----------------------------------------------------------------

public class NYStyleCheesePizza extends Pizza {

    public NYStyleCheesePizza() {
        name = "NY Style Sauce and Cheese Pizza";
        dough = "Thin Crust Dough"; // 薄餅皮
        sauce = "Marinara Sauce"; // 紅醬

        toppings.add("Grated Reggiano Cheese"); // 瑞吉起司的配料
    }
}

// ----------------------------------------------------------------

public class ChicagoStyleCheesePizza extends Pizza {

    public ChicagoStyleCheesePizza() {
        name = "Chicago Style Deep Dish Cheese Pizza";
        dough = "Extra Thick Crust Dough"; // 超厚餅皮
        sauce = "Plum Tomato Sauce"; // 番茄醬

        toppings.add("Shredded Mozzarella Cheese"); // 莫札瑞拉起司的配料
    }

    void cut() { // 芝加哥風味的Pizza覆寫cut方法，將Pizza切成方塊
        System.out.println("Cutting the pizza into square slices");
    }
}
```

9. 撰寫客戶端測試訂購 Pizza

```java
public class PizzaTestDrive {

    public static void main(String[] args) {
        // 建立兩個不同的Pizza店
        PizzaStore nyStore = new NYPizzaStore();
        PizzaStore chicagoStore = new ChicagoPizzaStore();

        // 用一家店來製作Ethan的訂單
        Pizza pizza = nyStore.orderPizza("cheese");
        System.out.println("Ethan ordered a " + pizza.getName() + "\n");

        // 用另一家店來製作Joel的訂單
        pizza = chicagoStore.orderPizza("cheese");
        System.out.println("Joel ordered a " + pizza.getName() + "\n");
    }
}
```

![建立者和產品視為平行的](./%E5%BB%BA%E7%AB%8B%E8%80%85%E5%92%8C%E7%94%A2%E5%93%81%E8%A6%96%E7%82%BA%E5%B9%B3%E8%A1%8C.png)

### 定義工廠模式

> [!TIP]
>
> **工廠方法模式**定義了一個創建物件的介面，但是他讓子類別決定想要實例化哪一個類別。工廠方法可以讓一個類別將實例化的動作推遲到子類別。

工廠方法模式可以讓我們將具體型態的實例化封裝起來。所以通常會聽到開發者說：「工廠方法模式可以讓子類別決定想要實例化的類別。」

### 沒有蠢問題

**Q: 如果 ConcreteCreator 只有一個時，工廠方法有什麼好處？**

A: 只有一個還是可以防止產品的實作和他的用法出現耦合，另外需要加入額外的產品或改變產品的實作，並不會影響你的 Creator。

**Q: 我們可以說 NYStylePizzaStore 和 ChicagoStylePizzaStore 都是用簡單工廠(Simple Factory)實作的嗎？**

A: 兩種做法很像，但用法不同。具體的 Pizza 店看起來很像 SimplePizzaFactory，但具體 Pizza 店繼承的類別定義的一個抽象方法 createPizza()，而 createPizza()方法是**由每一家店定義的**。在簡單工廠裡，工廠是與 PizzaStore 組合在一起的另一個物件。

**Q: 工廠方法和 Creator 類別一定要是抽象的嗎？**

A: 不是，可以定義預設的工廠方法來產生具體的產品。即使 Creator 沒有任何子類，還是可以製作產品。

**Q: 每一個具體建立者都要製作多個產品嗎？還是可以只製作一個？**

A: 我們製作的結構就是參數化工廠方法，可以根據傳來的參數製作多個物件。但是，工廠通常只會製作一個物件，而不使用參數。兩者型式都是有用的模式。

**Q: 用 String 型態來傳遞參數似乎不太安全，如果有人把 ClamPizza 寫成 CalmPizza？**

A: 對的，這會造成執行期錯誤，可以使用代表參數種類的物件、使用靜態常數、或使用 enum，確保編譯期可以抓到。

**Q: 簡單工廠和工廠方法還有沒有差異？**

A: 簡單工廠可以將物件的建立封裝起來，但是無法提供工廠方法的彈性，因為你無法改變你所建立的產品。

## 依賴反轉原則(Dependency Inversion Principle)

> [!IMPORTANT]
>
> 要依賴抽象，不要依賴具體類別

與「針對介面寫程式，不要針對實作寫程式」很像，但依賴反轉更強調**抽象**，提出高階的組件不應該依賴低階的組件，**兩者都要**依賴抽象。

### 第一版的 PizzaStore

-   因為 PizzaStore 直接建立這些 Pizza 物件，所以他依賴這些物件。如果這些 Pizza 的具體實作改變了，有可能就要修改 PizzaStore，所以我們說 PizzaStore**依賴**Pizza 具體實作。

![第一版PizzaStore依賴圖](./%E7%AC%AC%E4%B8%80%E7%89%88PizzaStore%E4%BE%9D%E8%B3%B4%E5%9C%96.png)

### 使用工廠模式後的 PizzaStore

-   主要問題是 PizzaStore 他依賴每一種 Pizza，因為 orderPizza 裡實例化具體型態。雖然建立一個抽象 Pizza，但我們仍建立許多具體的 Pizza。使用工廠模式，將 orderPizza 方法裡的實例化拿出來後，就可以解決依賴。此時 PizzaStore 只依賴 Pizza 介面，他是抽象類別，Pizza 介面也是抽象的，具體 Pizza 實作 Pizza 介面，所以依賴他。

![工廠模式後的PizzaStore依賴圖](./%E5%B7%A5%E5%BB%A0%E6%A8%A1%E5%BC%8F%E5%BE%8C%E7%9A%84PizzaStore%E4%BE%9D%E8%B3%B4%E5%9C%96.png)

### 以下方針可以避免設計違反依賴反轉原則

1. 任何變數都不應該保存具體類別的參考
    > 當你使用 new，你就會保存一個指向具體類別的參考，使用工廠來避免
2. 任何類別都不應該從具體類別衍生出來
    > 從具體類別衍生，就依賴一個具體類別
3. 任何方法都不應該覆寫基底類別的任何已實作的方法
    > 覆寫已實作的方法，就代表你的基底類別從一開始就不是真正的抽象。基底類的實作方法，是為了讓所有子類別共用的

**上面的原則是你要盡量遵守，不是不留餘地的死規則，目的是在違反原則時能立刻察覺，並知道何時有充分的理由可以違反原則**

-   我們經常不假思索的實例化 String 物件，有違反原則嗎？有，可以這樣做嗎？可以，為什麼？因為 String 非常不可能改變

## 工廠方法的例子

### JDK 的 java.util.Collection#iterator

-   Collection 介面就是一個抽象工廠，ArrayList 具體工廠
-   Iterator 介面是抽象產品，ListIterator 是抽象產品

![JDK中的應用](./JDK%E4%B8%AD%E7%9A%84%E6%87%89%E7%94%A8.png)

## Lambda 和 Enum 的寫法

### 只有 Lambda

:o:**優點**

1. 可以減少 Class 的數量

:x:**缺點**

1. 工廠需要多個引數時無法使用

```java
// ---------------------將工廠類改成Lambda-------------------
public class NYPizzaStore extends PizzaStore {

    // 改成Map對應
    private static final Map<String, Optional<Supplier<Pizza>>> map = new ConcurrentHashMap<>();

    static {
        map.put("NYStyleCheesePizza", Optional.of(NYStyleCheesePizza::new));
        map.put("NYStyleVeggiePizza", Optional.of(NYStyleVeggiePizza::new));
        map.put("NYStyleClamPizza", Optional.of(NYStyleClamPizza::new));
        map.put("NYStylePepperoniPizza", Optional.of(NYStylePepperoniPizza::new));
    }

    static Pizza createPizza(String item) { // 選擇哪一種紐約風味的Pizza
        return map.get(item)
                  .orElseThrow(() -> new IllegalArgumentException("No such item: " + item))
                  .get();
    }
}
```

### Lambda + Enum

:o:**優點**

1. Enum 可以不用寫 Optional

:x:**缺點**

1. 如果使用 Enum 則會造成選擇的邏輯無法集中在工廠裡面

```java
// -----------------------------建立ENUM類-----------------------------
public enum NYStylePizzaEnum {
    NYStyleCheesePizza(NYStyleCheesePizza::new),
    NYStyleVeggiePizza(NYStyleVeggiePizza::new),
    NYStyleClamPizza(NYStyleClamPizza::new),
    NYStylePepperoniPizza(NYStylePepperoniPizza::new);

    private final Supplier<Pizza> constructor;

    NYStylePizzaEnum(Supplier<Pizza> constructor) {
        this.constructor = constructor;
    }

    public Supplier<Pizza> getConstructor() {
        return constructor
    }
}

// -----------------------------將工廠改為使用ENUM類----------------------

public class NYPizzaStore extends PizzaStore {

    // 使用Enum來選擇，但如何判斷使用哪一個Enum的邏輯可能無法集中，會跑到工廠外部
    Pizza createPizza(NYStylePizzaEnum NYStylePizza) {
        return NYStylePizza.getConstructor().get();
    }
}
```
