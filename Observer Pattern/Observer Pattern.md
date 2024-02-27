# Observer Pattern (觀察者模式)

> **觀察者模式**是一種行為設計模式，定義物件之間的一對多依賴關係，當一個物件改變狀態時，依賴它的物件都會自動收到通知和更新。

### 觀察者模式結構

![觀察者模式結構](./%E8%A7%80%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F%E9%A1%9E%E5%88%A5%E5%9C%96.png)

### 觀察者模式的應用場景

1. **當一個對象狀態改變時，需要改變其他對象，或實際對象是事先未知的或動態變化的**
    > 圖形介面中，需要按鈕類允許客戶在按下按鈕時注入自己定義的程式碼，透過訂閱機制(addEventListener)，讓用戶，這樣當用戶按下時就會觸發程式碼
2. **應用中的一些對象必須觀察其他對象時，但僅能在有限時間內或特定情況下使用**
    > 訂閱列表是動態的，觀察者可以隨時加入或離開列表

### 優缺點

:o:**優點**

1. 開放封閉原則。無須修改 Subject 程式，就可以引入新的觀察者類別(如果是 Subject 介面則可輕鬆引入 Subject 類)
2. 可以在執行期間建立對象之間的關係

:x:**缺點**

1. 觀察者的通知順序是隨機的
2. 觀察者很多的時候，通知所有觀察者會很花時間
3. 如果觀察者和觀察目標之間有**循環依賴**，可能導致系統崩潰
4. 觀察者不知道其他觀察者的存在，依賴準則的定義或維護不當，容易引起錯誤的更新

## 原始的模型架構與需求

1. 我們有一個 weatherData，他儲存氣象局最新的測量數據(氣溫、濕度、溫度)，每當他改變的時候，我們的數值就要改變
2. 我們有多個畫面(目前的天氣、統計數據、天氣預測...)

![原始的模型架構](./%E5%8E%9F%E5%A7%8B%E7%9A%84%E6%A8%A1%E5%9E%8B%E6%9E%B6%E6%A7%8B.png)

### 錯誤寫法

```java
public void measurementsChanged() {
    float temperature = getTemperature();
    float humidity = getHumidity();
    float pressure = getPressure();

    /* 看起來下面這段式會改變的，需要封裝起來 */
    currentConditionsDisplay.update(temperature, humidity, pressure); // 傳入參數未來會越來越多
    statisticsDisplay.update(temperature, humidity, pressure); // 不一定每個畫面都需要所有觀測資料
    forecastDisplay.update(temperature, humidity, pressure);
    // 寫成具體實作後，我們就無法在不修改程式的情況下，加入或移除其他元素
}
```

> [!WARNING]
> 傳入參數未來可能會越來越多

> [!WARNING]
> 不一定每個畫面都需要所有觀測資料

> [!WARNING]
> 違反開放封閉原則，我們就無法在不修改程式的情況下，加入或移除其他元素

### 對象(SUBJECT) + 觀察者(OBSERVER) = 觀察者模式

> observer 有時又稱作 subscriber、listener

> [!NOTE] > **觀察者模式**定義物件之間的一對多依賴關係，當一個物件改變狀態時，依賴它的物件都會自動收到通知和更新。

![發布者訂閱者](./%E7%99%BC%E5%B8%83%E8%80%85%E8%A8%82%E9%96%B1%E8%80%85.png)

### 觀察者模式是鬆耦合

1. Subject 只知道觀察者實作了某個介面(Observer 介面)
2. 我們可以隨時加入新的觀察者
3. 如果要加入新的觀察者類型，我們完全不需要修改 Subject
4. 可以重複利用 Subject 或觀察者，又不會影響到對方
5. 修改 Subject 或觀察者，都不會影響到對方

> [!TIP]
> 努力為彼此互動的物件做出鬆耦合的設計

### 使用觀察模式後的設計

![設計氣象站](./%E8%A8%AD%E8%A8%88%E6%B0%A3%E8%B1%A1%E7%AB%99.png)

### 程式碼實作

## Pull Model / Pull Model

* Push model
  * 推送所有資料給Observer
  * Subject要知道Observer需要什麼，彈性較差
  * Observer會接收到不必要的資料
  * 好處是不需要保留Subject的引用
* Pull model
  * 提供必要的資料或其來源(如 data id 或 subject本身)給Observer，由Observer自行取得相對關資料
  * 每個Observer都要重新取得資料，效率較差
  * 壞處是需要保留Subject的引用

## 觀察者模式(Observer Pattern) vs 發布/訂閱模式(Publish/Subscribe Pattern)

|   比較   |               觀察者模式(Observer Pattern)                |        發布/訂閱模式(Publish/Subscribe Pattern)        |
| :------: | :-------------------------------------------------------: | :----------------------------------------------------: |
| 模式類別 |                      Design Pattern                       |                   Messaging Pattern                    |
| 知道對方 | 觀察者知道 Subject 的，Subject 也一直保持對觀察者進行記錄 | 發布者和訂閱者不知道對方的存在，只透過消息代理進行通訊 |
|  耦合性  |                         相對耦合                          |                     組件是鬆散耦合                     |
|   同步   |                  大多是同步(synchronous)                  |           大多是異步(asynchronous)(消息對列)           |

### 觀察者模式 + 中介者模式

> [!WARNING]
> 一般的觀察者模式Subject仍需要保留Observer的引用，無法真正解偶

![觀察者模式 + 中介者模式](./%E8%A7%80%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8Fplus%E4%B8%AD%E4%BB%8B%E8%80%85%E6%A8%A1%E5%BC%8F.png)

1. **封裝複雜的更新語意**。當目標與觀察者的依賴關係特別複雜時，可能需要一個維護這些關係的對象，稱作**更改管理器(ChangeManager)**。
2. ChangeManager是一個Mediator(中介者)模式的實例，通常是一個Singleleton(單例)

**ChangeManager有三個職責：**
1. 將一個Subject映射到他的觀察者，並提供一個介面來維護這個映射，這就不用由Subject來維護對觀察者的引用
2. 定義一個特定的更新策略
3. 根據一個目標請求，更新所有依賴於這個目標的觀察者

![ChangeManager](./%E8%A7%80%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8Fplus%E4%B8%AD%E4%BB%8B%E8%80%85%E6%A8%A1%E5%BC%8F.gif)

* 當一個觀察者觀察多個目標時，DAGChangeManager要更好用一些，他可以保證觀察者僅接受一個更新，而不會接受到多個冗余的更新。
* 當不存在重複更新時，SimpleChangeManager。


### 發布/訂閱模式

![發布/訂閱模式](./%E7%99%BC%E4%BD%88%E8%A8%82%E9%96%B1%E6%A8%A1%E5%BC%8F.png?raw=true)

1. 模組的解偶
    > 發佈者(Publisher)和訂閱者(Subscriber)之間，透過中間人(broker)或Message/Event Bus來解偶
    > 就像訂閱某個粉專，訂閱者不需要知道發文的小編是誰

2. 時間的解偶
    > 發佈訊息時，訂閱者不一定在線上，採用先存再送(store-and-forward)的機制