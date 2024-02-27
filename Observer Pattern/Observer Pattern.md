# Observer Pattern (觀察者模式)

## 觀察者模式結構



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

### 鬆耦合

1. Subject只知道觀察者實作了某個介面(Observer介面)
2. 我們可以隨時加入新的觀察者
3. 如果要加入新的觀察者類型，我們完全不需要修改Subject
4. 可以重複利用Subject或觀察者，又不會影響到對方
5. 修改Subject或觀察者，都不會影響到對方

> [!TIP]
> 努力為彼此互動的物件做出鬆耦合的設計


