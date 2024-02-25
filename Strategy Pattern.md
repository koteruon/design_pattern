# Straegy Pattern

## 原始的模型架構

有一個鴨子模擬遊戲，稱作SimUDuck，裡面的定義了一個Duck超類別，讓所有品種的鴨子來繼承他。

![原始APP的樣貌](https://raw.githubusercontent.com/koteruon/design_pattern/main/Strategy%20Pattern/%E5%8E%9F%E5%A7%8BAPP%E7%9A%84%E6%A8%A3%E8%B2%8C.png?token=GHSAT0AAAAAACOMRJHA6BURSAIU4ONZ3NBIZO3EX6Q)

## 需求增加一個會飛的行為

### 在抽象類別中增加一個Fly()行為

* 缺點
