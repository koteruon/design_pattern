# Decorator Pattern (裝飾器模式)

> **裝飾器模式**是一種結構型設計模式，可以動態地為物件附加額外的職責。

### 裝飾器模式結構

![裝飾器模式結構](./%E8%A3%9D%E9%A3%BE%E5%99%A8%E6%A8%A1%E5%BC%8F%E6%9E%B6%E6%A7%8B.png)

### 裝飾器模式的應用場景

1. **如果你希望在無需修改程式碼的情況下即可使用對象，並且希望在運行時為對象增加額外的"行為"**
    > 裝飾器模式能將業務邏輯組織成層次結構，可以為各層創建一個裝飾，在運行期間將各種不同的邏輯組合成對象。這些對象都遵循通用介面，客戶端地程式碼能以相同方式使用對象。
2. **如果用繼承來擴展對象行為的方案難以實現**
    > 許多程式使用final限制類的進一步擴展，復用最終類已有的行為的唯一方法就是裝飾器模式：用封裝器對其進行封裝。

### 優缺點

:o:**優點**

1. 無須創建新子類即可擴展對象的行為
2. 可以在運行期間添加或刪除對象的功能
3. 可以用多個裝飾封裝對象來組合多種行為
4. 單一職責原則。可以將實現了許多不同行為的一個大類，拆分成許多較小的類

:x:**缺點**

1. 在裝飾器stack中刪除特定的裝飾器比較困難
2. 實現行為不受裝飾stack順序影響的裝飾比較困難
3. 各層的初始化配置程式碼看上去可能會很糟糕
4. 會產生很多小對象，同時也會產生很多具裝飾類，會增加系統的複雜度和學習與理解難度，有時需要工廠模式協助

## 原始的模型架構

1. 創立初期，使用抽象介面定義飲料(Beverage)，咖啡店的所有飲料都繼承他。
2. description用來保存飲料的敘述。
3. cost()由子類別實作，用來回傳飲料的價格。

![原始的模型架構](./%E5%8E%9F%E5%A7%8B%E7%9A%84%E6%A8%A1%E5%9E%8B.png)

### 遇到的需求與問題

1. 因為飲料增加了調味料，如牛奶、豆漿、摩卡、奶泡。根據不同的調味品加收不同的費用，造成**類別大爆炸**。

![子類爆炸](./%E5%AD%90%E9%A1%9E%E7%88%86%E7%82%B8.png)

### 錯誤方法

1. 將調味料放入飲料類別，透過has去判斷是否有調味料，在每一個子類的cost()取得調味品，並計算加上調味品的價格

![錯誤方法](./%E9%8C%AF%E8%AA%A4%E6%96%B9%E6%B3%95.png)

> [!WARNING]
> 違反開放/封閉原則。未來調味料越多，Beverage和子類的cost就必須跟著修改。

## 開放/封閉原則

我們的目標是讓類別容易擴展，藉以納入新行為，但是不能修改既有的程式碼。實現這個目標有什麼好處？這種設計不但有因應改變的韌性，也有足夠的彈性，可以接納新功能，來滿足不斷改變的需求。

> [!TIP]
> 類別應該歡迎擴展，但拒絕修改
>
> Classes should be open for extension, but closed for modification.
>
> 類別應該對擴展開放，對修改封閉

### 沒有蠢問題

**Q: 該怎麼讓設計的每一個部分都遵守開放/封閉原則？**

A: 這通常不可能做到。一般來說，我們沒有那麼多資源可以把設計的每一個細節都做成這樣。遵守開放/封閉原則通常會引入新一層的抽象，讓程式更複雜。應該把注意力放在最有可能改變的地方，並且在那裡實施這些規則。

> [!NOTE]
> 請謹慎地選擇需要擴展的部分，**到處**採用開放/封閉原則不但浪費，也沒有必要，甚至可能寫出複雜的、難以理解的程式。

## 裝飾器概念

![裝飾器概念](./%E8%A3%9D%E9%A3%BE%E5%99%A8%E6%A6%82%E5%BF%B5.png)

![裝飾器例子類別圖](./%E8%A3%9D%E9%A3%BE%E5%99%A8%E4%BE%8B%E5%AD%90%E9%A1%9E%E5%88%A5%E5%9C%96.png?raw=true)

**Q: CondimentDecorator繼承Beverage類別，這不是繼承關係嗎？**

A: 為了讓裝飾器和被他裝飾的物件有相同的型態，使用繼承*讓他們有相同的型態*，這裡的繼承*不是為了獲得行為*。

**Q: 行為是怎麼加入的？**

A: 我們不是藉著*繼承*超類別來獲得新行為，而是藉著將物件*組合*起來。因為依靠繼承，行為只能在編譯其決定，固定不變。但組合可以在執行期間，動態隨意混合搭配裝飾器。

**Q: 為什麼不把Beverage設計成介面？**

A: 原本就有一個抽象的Beverage，當然可以使用介面，但通常為了避免修改既有的程式碼，所以不會在抽象類別沒有任何問題時修正他。

## 程式碼

1. 不需要修改原本的抽象飲料類別

```java
public abstract class Beverage {
    String description = "Unknown Beverage";

    public String getDescription() { // 寫好通用的方法
        return description;
    }

    public abstract double cost(); // 在子類別裡面實作
}
```

2. 為了可以和Beverage互換(包裝)，我們繼承Beverage類別

```java
public abstract class CondimentDecorator extends Beverage {
    Beverage beverage; // 我們使用Beverage超型態來引用，好讓Decorator可以包裝任何一種飲料
    public abstract String getDescription(); // 讓調味品都可以重新實作getDescription
}
```

3. 撰寫飲料基底的類別，繼承Beverage，因為它是一種飲料

```java
public class HouseBlend extends Beverage {
    public HouseBlend() {
        description = "House Blend Coffee";
    }

    public double cost() {
        return .89; // 不需要在這個類別裡加上調味品，因此回傳原本飲料的價錢
    }
}

// -------------------------------------------

public class Espresso extends Beverage {

    public Espresso() {
        description = "Espresso";
    }

    public double cost() {
        return 1.99; // 不需要在這個類別裡加上調味品，因此回傳原本飲料的價錢
    }
}
```

4. 撰寫調味品，繼承CondimentDecorator，用Beverage的參考來實例化

```java
public class Mocha extends CondimentDecorator {
    public Mocha(Beverage beverage) { // 保存被包裝的飲料(or調味品)
        this.beverage = beverage;
    }

    public String getDescription() {
        return beverage.getDescription() + ", Mocha"; // 希望description不只包含飲料，也包含被裝飾的敘述
    }

    public double cost() {
        return .20 + beverage.cost(); // 計算添加Mocha的飲料的價格，先呼叫被裝飾的物件，再將結果加上Mocha的價格。
    }
}
```

5. 最後於客戶端進行測試

```java
public class StarbuzzCoffee {

    public static void main(String args[]) {
        Beverage beverage = new Espresso();
        System.out.println(beverage.getDescription() + " $" + beverage.cost()); // 不添加調味料

        Beverage beverage2 = new DarkRoast();
        beverage2 = new Mocha(beverage2); // 可以動態決定要加多少個，多少種類都行
        beverage2 = new Mocha(beverage2);
        beverage2 = new Whip(beverage2);
        System.out.println(beverage2.getDescription() + " $" + beverage2.cost()); // 添加調味料
    }
}
```

### 沒有蠢問題

**Q: 如果測試某個具體組件來做某件事情的程式，如：HouseBlend來進行打折，當HouseBlend包在裝飾器裡面之後，那種程式就失效了**

A: 當程式*依賴具體組件型態*時，裝飾器會破壞那段程式碼，但只要針對*抽象組件型態寫程式*，那麼裝飾器對程式碼來說仍然透明。然而，一旦針對*具體的組件寫程式*，就要重新思考應用程式的設計，以及裝飾器的使用了。

## 更多實際上的例子

1. java.io中的InputStream、FilterInputStream、BufferedInputStream、ZipInputStream

2. spring中的

3. notify中有多個