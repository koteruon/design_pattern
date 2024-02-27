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

1. 開放封閉原則。無須修改Subject程式，就可以引入新的觀察者類別(如果是Subject介面則可輕鬆引入Subject類)
2. 可以在執行期間建立對象之間的關係

:x:**缺點**

1. 觀察者的通知順序是隨機的

## 原始的模型架構與需求

1. 我們有一個weatherData，他儲存氣象局最新的測量數據(氣溫、濕度、溫度)，每當他改變的時候，我們的數值就要改變
2. 我們有多個畫面(目前的天氣、統計數據、天氣預測...)

![原始的模型架構](./%E5%8E%9F%E5%A7%8B%E7%9A%84%E6%A8%A1%E5%9E%8B%E6%9E%B6%E6%A7%8B.png)

### 錯誤寫法

```java
public void measurementsChanged() {
    float temperature = getTemperature;
    float humidity = getHumidity;
    float pressure = getPressure;

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
> observer有時又稱作subscriber、listener

> [!NOTE]
> **觀察者模式**定義物件之間的一對多依賴關係，當一個物件改變狀態時，依賴它的物件都會自動收到通知和更新。

![發布者訂閱者](./%E7%99%BC%E5%B8%83%E8%80%85%E8%A8%82%E9%96%B1%E8%80%85.png)

### 觀察者模式是鬆耦合

1. Subject只知道觀察者實作了某個介面(Observer介面)
2. 我們可以隨時加入新的觀察者
3. 如果要加入新的觀察者類型，我們完全不需要修改Subject
4. 可以重複利用Subject或觀察者，又不會影響到對方
5. 修改Subject或觀察者，都不會影響到對方

> [!TIP]
> 努力為彼此互動的物件做出鬆耦合的設計


