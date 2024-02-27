# Straegy Pattern (策略模式)

> **策略模式**是一種行為設計模式，他能讓你定義一系列算法，並將每種算法分別放入獨立的類中，以使算法的對象能夠相互替換。

### 策略模式結構

![策略模式結構](./%E6%A8%A1%E5%BC%8F%E7%B5%90%E6%A7%8B.png)

### 策略模式適合的應用場景

1. **當你想使用各種不同的算法，並且希望能在運行期間切換算法時**
	> 策略模式可以關聯不同的子對象，從而間接在運行時更改對象行為
2. **當有許多僅在執行某些行為時略有不同的相似類時**
	> 策略模式可以將不同的行為抽象到一個獨立類結構中，並將原始類組合成同一個
3. **如果算法在上下文的邏輯中不是特別重要，可以將類的業務邏輯和他的算法細節隔離出來**
	> 策略模式能將各種算法的代碼、內部數據、依賴關係，跟其他代碼分開來
4. **當類中使用了複雜條件運算，以在同一算法的不同變體中切換**
	> 策略模式將所有繼承相同接口的算法抽取至獨立類中，因此不需要條件語句。原始對象也不實現所有算法的變體，而是將執行工作委託給其中一個獨立算法的對象。

### 優缺點

:o:**優點**

1. 在運行時切換對象的算法
2. 將算法的實現和算法的代碼隔離開來
3. 使用組合代替繼承
4. 開放封閉原則。無須對上下文修改，就可以引入新策略

:x:**缺點**

1. 若算法極少發生改變，沒有理由引入新的類和介面，會使程序更為複雜
2. 客戶端必須知道策略間的不同，因為要選擇合適的策略
3. [許多程式語言支持函數類型](./Strategy%20Pattern.md#java-使用-lamda-簡化策略模式)，允許在匿名函數中實現不同版本的算法。這些函數就和使用策略對象完全相同，無須借助額外的類和介面來保持程式乾淨。

## 原始的模型架構

有一個鴨子模擬遊戲，稱作SimUDuck，裡面的定義了一個Duck超類別，讓所有品種的鴨子來繼承他。

![原始APP的樣貌](./%E5%8E%9F%E5%A7%8BAPP%E7%9A%84%E6%A8%A3%E8%B2%8C.png)

## 需求增加一個會飛的行為

### 在抽象類別中增加一個Fly()行為

![抽象類別增加一個fly](./%E6%8A%BD%E8%B1%A1%E9%A1%9E%E5%88%A5%E5%A2%9E%E5%8A%A0%E4%B8%80%E5%80%8Bfly.png)

**缺點**

1. 有可能有一種鴨(橡皮鴨)不會飛
2. 多一個子類又必須複寫一次fly這個行為(改為不會飛)

> [!WARNING]
> 局部修改程式碼，卻造成非局部性的影響！

### 使用介面增加一個Flyable、Quackable

![介面增加Flyable和Quackable](./%E4%BB%8B%E9%9D%A2%E5%A2%9E%E5%8A%A0Flyable%E5%92%8CQuackable.png)

**缺點**

1. 如果想要改fly這個行為，所有的子類別都要改

> [!WARNING]
> 程式碼無法重複利用

---

## 分析原因

1. 子類別裡面的鴨子行為會不斷改變
2. 繼承的效果不好，介面又無法實作程式碼

> [!TIP]
> 找出應用程式中會變得部分，把他們和不會變的部分區分隔開

> [!NOTE]
> 把會變得部分封裝起來，如此一來，你就可以修改或擴展會變得部分，同時不會影響不變的部分

### 把會變的部分和不變的部分分開

**Duck類別裡有fly()和quack()會隨著鴨自不同而改變，所以抽出Duck類別，並建立一組類別來代表那些行為。Duck就將飛行和鳴叫的行為** *委託(delegate)* **出去**

![針對介面寫程式(實作Duck行為)](./%E9%87%9D%E5%B0%8D%E4%BB%8B%E9%9D%A2%E5%AF%AB%E7%A8%8B%E5%BC%8F(%E5%AF%A6%E4%BD%9CDuck%E8%A1%8C%E7%82%BA).png)

> [!TIP]
> 針對介面寫程式，而不是針對實作寫程式

針對介面寫程式 == 針對超型態(supertype)寫程式 == 用來宣告變數的型態應該是超型態(*通常是抽象類別或介面*)

```java
// 針對實作寫程式
Dog d = new Dog();
d.bark()

// 針對介面/超型態寫程式
Animal animal = new Dog();
animal.makeSound()

// 更好的方法是在執行期指派具體的實作物件，而不是用固定的程式來實例化子型態
Animal a = getAnimal()
a.makeSound() // 我們不知道animal的子型態到底是什麼，只知道他如何回應
```

**好處**

1. 可以讓其他物件型態重複使用飛行和鳴叫行為，因為這些行為沒有被埋在Duck類別裡
2. 也可以加入新行為，而且不需要修改既有的行為類別，或修改使用飛行行為的Duck類別

---

## 沒有蠢問題

**Q: 先寫好應用程式，看看哪些地方會改變，再回過頭來隔離並封裝會變的地方嗎？**

A: 不一定，在設計應用程式的時候，通常可以預料哪些地方會改變，提前加入處理他的彈性機制。

**Q: 是不是也要把Duck做成介面？**

A: 在這個例子不用。繼承共同的屬性和方法是有好處的(不變的地方)，同時我們也已經把會變得部分移開了。

**Q: 只有一個行為的類別有點奇怪，類別不是用來表示某一種「東西」嗎？**

A: 在物件導向中，類別通常代表兼具狀態(實例變數)的方法的東西。在這個例子裡，那個東西碰巧是一種行為，但是，行為同樣有狀態和方法，飛行行為可能用實例變數來代表飛行行為的屬性(每分鐘揮動幾下翅膀、最大飛行高度...等)

---

## 綜觀封裝行為

![綜觀封裝行為](./%E7%B6%9C%E8%A7%80%E5%B0%81%E8%A3%9D%E8%A1%8C%E7%82%BA.png)

### HAS-A 有時比 IS-A 還要好

每隻鴨子都有FlyBehavior與QuackBehavior，鴨子會將**飛行**和**鳴叫**委託給他們。

把這兩個類別結合起來就是**組合(composition)**，鴨子的行為不是繼承而來，而是和行為物件組合起來獲得的。

> [!TIP]
> 多用組合，少用繼承

### 程式碼

1. 創建abstract class Duck，並宣告兩個行為介面型態的參考變數，鴨子子類都會繼承他們。

```java
public abstract class Duck {
	FlyBehavior flyBehavior; //行為介面型態的參考變數
	QuackBehavior quackBehavior; //行為介面型態的參考變數

	public Duck() {
	}

	public void setFlyBehavior(FlyBehavior fb) { // 動態改變鴨子的行為
		flyBehavior = fb;
	}

	public void setQuackBehavior(QuackBehavior qb) { // 動態改變鴨子的行為
		quackBehavior = qb;
	}

	abstract void display();

	public void performFly() {
		flyBehavior.fly(); // 委託給行為類別
	}

	public void performQuack() {
		quackBehavior.quack(); // 委託給行為類別
	}

	public void swim() {
		System.out.println("All ducks float, even decoys!");
	}
}
```

2. FlyBehavior介面，並建立兩個行為實作類別

```java
public interface FlyBehavior {
	public void fly();
}

//----------------------------------------------------------------

public class FlyWithWings implements FlyBehavior {
	public void fly() {
		System.out.println("I'm flying!!");
	}
}

//----------------------------------------------------------------

public class FlyNoWay implements FlyBehavior {
	public void fly() {
		System.out.println("I can't fly");
	}
}

//----------------------------------------------------------------

public class FlyRocketPowered implements FlyBehavior { // 新增火箭的飛行模式
	public void fly() {
		System.out.println("I'm flying with a rocket");
	}
}


```

3. QuackBehavior介面，創建三個行為實作類別

```java
public interface QuackBehavior {
	public void quack();
}

//----------------------------------------------------------------

public class Quack implements QuackBehavior {
	public void quack() {
		System.out.println("Quack");
	}
}

//----------------------------------------------------------------

public class MuteQuack implements QuackBehavior {
	public void quack() {
		System.out.println("<< Silence >>");
	}
}

//----------------------------------------------------------------

public class Squeak implements QuackBehavior {
	public void quack() {
		System.out.println("Squeak");
	}
}
```

4. 製作新的Duck型態(ModelDuck)

```java
public class ModelDuck extends Duck {
	public ModelDuck() {
		flyBehavior = new FlyNoWay(); // 一開始的模型鴨是陸棲的，他不會飛
		quackBehavior = new Quack();
	}

	public void display() {
		System.out.println("I'm a model duck");
	}
}

```

5. 測試原本的ModelDuck，並動態加上火箭的飛行功能

```java
public class MiniDuckSimulator {
	public static void main(String[] args) {
		Duck model = new ModelDuck();
		model.performFly();
		model.setFlyBehavior(new FlyRocketPowered()); // 動態設定飛行方式
		model.performFly();
	}
}

```

**要在執行期間改變鴨子的行為，你只需要呼叫鴨子的行為setter即可**

---

## Java 使用 Lamda 簡化策略模式

1. 定義Context記錄指向算法的實例

```java
public class Context {
	private final Operation operation;

	public Context(Operation operation) {
		this.operation = operation;
	}

	public int getResult(int num1, int num2) {
		return operation.doSomething(num1, num2);
	}
}

```

2. 定義函數是介面，注意要加上**@FunctionalInterface**

```java
// 函數式介面就是只有一個抽象方法的介面
// 在策略模式中的算法，就是一個函數式介面
// 此處標註了介面為一個函數式介面，如果介面定義了多個抽象方法，會在編譯時期報錯

@FunctionalInterface
public interface Operation {
	int doSomething(int num1, int num2);
}
```

3. 測試利用不同的算法得到不同的結果

```java
public class Test {
	public static void main(String[] args) {
		Context context = new Context(((num1, num2) -> num1 + num2)) // 設定加法算法
		System.out.println("1 + 1 = " + context.getResult(1,1)); // 1 + 1 = 2
		Context context = new Context(((num1, num2) -> num1 - num2)) // 設定加法算法
		System.out.println("1 - 1 = " + context.getResult(1,1)); // 1 - 1 = 0
		Context context = new Context(((num1, num2) -> num1 * num2)) // 設定加法算法
		System.out.println("1 * 1 = " + context.getResult(1,1)); // 1 * 1 = 1
		Context context = new Context(((num1, num2) -> num1 / num2)) // 設定加法算法
		System.out.println("1 / 1 = " + context.getResult(1,1)); // 1 / 1 = 1
	}
}
```



