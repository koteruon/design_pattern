# Straegy Pattern

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

> [!TIP]
> 把會變得部分封裝起來，如此一來，你就可以修改或擴展會變得部分，同時不會影響不變的部分

### 把會變的部分和不變的部分分開

