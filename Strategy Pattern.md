# Straegy Pattern (策略模式)

## 原始的模型架構

有一個鴨子模擬遊戲，稱作SimUDuck，裡面的定義了一個Duck超類別，讓所有品種的鴨子來繼承他。

![原始APP的樣貌](https://github.com/koteruon/design_pattern/blob/main/Strategy%20Pattern/%E5%8E%9F%E5%A7%8BAPP%E7%9A%84%E6%A8%A3%E8%B2%8C.png)

## 需求增加一個會飛的行為

### 在抽象類別中增加一個Fly()行為

![抽象類別增加一個fly](https://github.com/koteruon/design_pattern/blob/main/Strategy%20Pattern/%E6%8A%BD%E8%B1%A1%E9%A1%9E%E5%88%A5%E5%A2%9E%E5%8A%A0%E4%B8%80%E5%80%8Bfly.png)

**缺點**

1. 有可能有一種鴨(橡皮鴨)不會飛
2. 多一個子類又必須複寫一次fly這個行為(改為不會飛)

> [!WARNING]
> 局部修改程式碼，卻造成非局部性的影響！

### 使用介面增加一個Flyable、Quackable

![介面增加Flyable和Quackable](https://github.com/koteruon/design_pattern/blob/main/Strategy%20Pattern/%E4%BB%8B%E9%9D%A2%E5%A2%9E%E5%8A%A0Flyable%E5%92%8CQuackable.png)

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

**Duck類別裡有fly()和quack()會隨著鴨自不同而改變，所以抽出Duck類別，並建立一組類別來代表那些行為。**

![針對介面寫程式(實作Duck行為)](https://github.com/koteruon/design_pattern/blob/main/Strategy%20Pattern/%E9%87%9D%E5%B0%8D%E4%BB%8B%E9%9D%A2%E5%AF%AB%E7%A8%8B%E5%BC%8F(%E5%AF%A6%E4%BD%9CDuck%E8%A1%8C%E7%82%BA).png)

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

Q: 先寫好應用程式，看看哪些地方會改變，再回過頭來隔離並封裝會變的地方嗎？
A: 不一定，在設計應用程式的時候，通常可以預料哪些地方會改變，提前加入處理他的彈性機制。

Q: 是不是也要把Duck做成介面？
A: 在這個例子不用。繼承共同的屬性和方法是有好處的(不變的地方)，同時我們也已經把會變得部分移開了。

Q: 只有一個行為的類別有點奇怪，類別不是用來表示某一種「東西」嗎？
A: 在物件導向中，類別通常代表兼具狀態(實例變數)的方法的東西。在這個例子裡，那個東西碰巧是一種行為，但是，行為同樣有狀態和方法，飛行行為可能用實例變數來代表飛行行為的屬性(每分鐘揮動幾下翅膀、最大飛行高度...等)

---

## 綜觀封裝行為

![綜觀封裝行為](https://github.com/koteruon/design_pattern/blob/main/Strategy%20Pattern/%E7%B6%9C%E8%A7%80%E5%B0%81%E8%A3%9D%E8%A1%8C%E7%82%BA.png)

1. 中間為封裝起來的飛行行為