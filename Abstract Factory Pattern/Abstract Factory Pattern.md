# Abstract Factory Pattern (抽象工廠模式) (Kit Pattern)

> **抽象工廠模式**是一種創建型設計模式，提供一個介面來建立相關或相依的物件家族，而不需要指定具體類別

### 抽象工廠模式結構

![抽象工廠模式結構](./%E6%8A%BD%E8%B1%A1%E5%B7%A5%E5%BB%A0%E6%A8%A1%E5%BC%8F%E7%B5%90%E6%A7%8B.png)

### 抽象工廠模式的應用場景

1. **如果程式碼需要與多個不同系列的相關產品溝通，但由於無法提前獲取相關訊息，或者出於對未來擴展性的考慮，你不希望程式碼基於產品的具體類進行建構**
    > 抽象工廠提供了一個介面，可以用於創建每個系列產品的對象。只需要程式碼通過介面創建對象，就不會生成與應用程式已生成的產品類型不一致的產品。
2. **如果你有一個基於一組抽象方法的類，且其主要功能因此變得不明確，那麼在這種情況下可以考慮使用抽象工廠**
    > 在設計良好的程式碼中，每個類**僅負責一件事**。如果一個類與多種類型產品溝通，就可以考慮將工廠方法抽取到**獨立的工廠類**或具備完整功能的**抽象工廠類**中。

### 優缺點

:o:**優點**

1. 可以確保同一工廠生產的產品相互匹配
2. 可以避免客戶端與具體產品程式碼的耦合
3. 單一職責原則。可以將產品生成程式碼抽取到同一個位置，使代碼易於維護
4. 開放封閉原則。向應用程式中引入新產品時，無須修改客戶端程式碼

:x:**缺點**

1. 由於採用模式需要向應用中引入眾多介面和類，程式碼可能會比之前更加複雜

## 抽象工廠模式(Abstract Factory Pattern)

### 原始的模型架構

-   以下的情境是基於工廠模式後而來[工廠模式](../Factory%20Method%20Pattern/Factory%20Method%20Pattern.md)

### 遇到的需求與問題

1.  Pizza 店的製作流程被統一後，但 Pizza 產品內部的食材卻因地區不同，而且 NYStyleCheesePizza 與 ChicagoStyleCheesePizza 只差在地區性食材的地方，Pizza 的作法是相同的(餅皮+醬料+起司)，都有相同的準備步驟，只是使用不同的食材。

```java
public class NYStyleCheesePizza extends Pizza {

    public NYStyleCheesePizza() {
        name = "NY Style Sauce and Cheese Pizza";
        dough = "Thin Crust Dough"; // 餅皮(只差在地區性食材不同)
        sauce = "Marinara Sauce"; // 醬料(只差在地區性食材不同)

        toppings.add("Grated Reggiano Cheese"); // 起司(只差在地區性食材不同)
    }
}

// ----------------------------------------------------------------

public class ChicagoStyleCheesePizza extends Pizza {

    public ChicagoStyleCheesePizza() {
        name = "Chicago Style Deep Dish Cheese Pizza";
        dough = "Extra Thick Crust Dough"; // 餅皮(只差在地區性食材不同)
        sauce = "Plum Tomato Sauce"; // 醬料(只差在地區性食材不同)

        toppings.add("Shredded Mozzarella Cheese"); // 起司(只差在地區性食材不同)
    }

    void cut() {
        System.out.println("Cutting the pizza into square slices");
    }
}
```

### (抽象工廠)為食材製作工廠

> [!TIP]
>
> **工廠方法模式**定義了一個創建物件的介面，但是他讓子類別決定想要實例化哪一個類別。工廠方法可以讓一個類別將實例化的動作推遲到子類別。

1.  首先定義介面的抽象工廠，實例**不同地區的食材工廠**來針對**不同地區風味的食材**做製造

```java
public interface PizzaIngredientFactory {

    // 介面裡面，我們為每一種食材定義一個create方法
    // 每一種食材一個類別
    public Dough createDough();
    public Sauce createSauce();
    public Cheese createCheese();
    public Veggies[] createVeggies();
    public Pepperoni createPepperoni();
    public Clams createClam();
}

// ----------------------------------------------------------------

public class NYPizzaIngredientFactory implements PizzaIngredientFactory { // 紐約風味食材工廠實作所有食材工廠的介面

    /* 為每一種食材家族裡的每一種食材建立紐約風味 */
    public Dough createDough() {
        return new ThinCrustDough();
    }

    public Sauce createSauce() {
        return new MarinaraSauce();
    }

    public Cheese createCheese() {
        return new ReggianoCheese();
    }

    // 這裡將蔬菜寫死，我們也可以採取更精巧的作法
    public Veggies[] createVeggies() {
        Veggies veggies[] = { new Garlic(), new Onion(), new Mushroom(), new RedPepper() };
        return veggies;
    }

    // 紐約和芝加哥風味都使用相同的食材
    public Pepperoni createPepperoni() {
        return new SlicedPepperoni();
    }

    // 紐約靠海，所以他有新鮮的。
    public Clams createClam() {
        return new FreshClams();
    }
}
```

12. 修改 Pizza 類，讓食材工廠生產食材即可

```java
public abstract class Pizza {
	String name;

    /* 每一種Pizza都保存一組食材，在準備時使用 */
	Dough dough;
	Sauce sauce;
	Veggies veggies[];
	Cheese cheese;
	Pepperoni pepperoni;
	Clams clam;

    // 把prepare宣告成抽象的。我們將在這裡收集Pizza
	abstract void prepare();

	void bake() {
		System.out.println("Bake for 25 minutes at 350");
	}

	void cut() {
		System.out.println("Cutting the pizza into diagonal slices");
	}

	void box() {
		System.out.println("Place pizza in official PizzaStore box");
	}

	void setName(String name) {
		this.name = name;
	}

	String getName() {
		return name;
	}
}
```

## 比較工廠方法和抽象工廠

|   比較   |               觀察者模式(Observer Pattern)                |        發布/訂閱模式(Publish/Subscribe Pattern)        |
| :------: | :-------------------------------------------------------: | :----------------------------------------------------: |
| 模式類別 |                      Design Pattern                       |                   Messaging Pattern                    |
| 知道對方 | 觀察者知道 Subject 的，Subject 也一直保持對觀察者進行記錄 | 發布者和訂閱者不知道對方的存在，只透過消息代理進行通訊 |
|  耦合性  |                         相對耦合                          |                     組件是鬆散耦合                     |
|   同步   |                  大多是同步(synchronous)                  |           大多是異步(asynchronous)(消息對列)           |
