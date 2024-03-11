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
> *實作介面* **不一定**代表 *在類別宣告式裡面使用implements來編寫一個實作了Java介面的類別* 。廣義來說，讓一個具體的類別實作超型態(可能是抽象類別或是介面)的方法，仍然可以視為 *實作* 那個超型態的 *介面*。

## 工廠方法(Factory Method)

5. 希望加盟Pizza店，因此創建兩個NYPizzaFactory和ChicagoPizzaFactory，製作不同風味的pizza。

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
> 加盟的nyStore和chicagoStore開始採取自創的流程，因此我們需要把變得部分移出，利用抽象物件保留不變的部分(Pizza店和Pizza的製作過程綁再一起)，同時保持一些彈性由子類自己決定

### 為pizza店設計框架、讓子類做決定

6. 在PizzaStore類別裡，將所有的pizza製作動作局部化(localize)，同時讓連鎖店可以自由地製作地區風味

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

    abstract Pizza createPizza(String item); // 現在「工廠方法」是在PizzaStore裡面的抽象方法，也強迫所有子類必須實現這個方法
}
```

> [!NOTE]
> PizzaStore就是工廠，子類就是具體工廠，從SimplePizzaFactory的角色由PizzaStore的子類承擔

7. 創建兩間Pizza店，NYPizzaStore和ChicagoStylePizzaStore，他們都繼承PizzaStore，子類別會用自己的createPizza方法製作地區風味pizza

```java
public class NYPizzaStore extends PizzaStore {

    Pizza createPizza(String item) { // 想要紐約風味的pizza
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

    Pizza createPizza(String item) { // 想要芝加哥風味的pizza
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