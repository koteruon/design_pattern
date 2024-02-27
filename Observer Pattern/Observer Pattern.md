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

1. 針對 Subject 建立介面，包和 register、remove、notify
2. 針對 Observer 建立介面，讓所有觀察者實作

```java
public interface Subject {
	public void registerObserver(Observer o); // 接收Observer引數，註冊Observer
	public void removeObserver(Observer o); // 接收Observer引數，移除Observer
	public void notifyObservers(); // Subject改變時，這個方法會被呼叫，藉此通知觀察者
}

// ----------------------------------------------------------------

public interface Observer {
	public void update(float temp, float humidity, float pressure); // 這些測量數據改變時，Observer從Subject取得的狀態值
}

// ----------------------------------------------------------------

public interface DisplayElement {
	public void display(); // 想要顯示的元素時，呼叫他
}
```

3. 讓 WeatherData 實作 Subject 介面

```java
public class WeatherData implements Subject {
	private List<Observer> observers; // 保存觀察者的引用對象
	private float temperature;
	private float humidity;
	private float pressure;

	public WeatherData() {
		observers = new ArrayList<Observer>();
	}

	public void registerObserver(Observer o) {
		observers.add(o); //註冊時加入到List的最後
	}

	public void removeObserver(Observer o) {
		observers.remove(o); // 退出時從List中移除
	}

	public void notifyObservers() {
		for (Observer observer : observers) { // 所有觀察者皆實作update方法，在這裡將最新狀態傳給每一個觀察者
			observer.update(temperature, humidity, pressure);
		}
	}

	public void measurementsChanged() {
		notifyObservers(); // 當氣象站取得新的資料時，通知Observer
	}

    public void setMeasurements(float temperature, float humidity, float pressure) {
        // 測試用，假設數據更新了
		this.temperature = temperature;
		this.humidity = humidity;
		this.pressure = pressure;
		measurementsChanged();
	}

	public float getTemperature() {
		return temperature; // pull model使用
	}

	public float getHumidity() {
		return humidity; // pull model使用
	}

	public float getPressure() {
		return pressure; // pull model使用
	}

}

```

4. 取其中一個顯示元素當作例子，他實作 Observer 介面，可以從 WeatherData 物件收到變更。

```java
public class CurrentConditionsDisplay implements Observer, DisplayElement {
	private float temperature;
	private float humidity;
	private WeatherData weatherData; // 未來想要退出訂閱時可以使用到(不一定要)

	public CurrentConditionsDisplay(WeatherData weatherData) {
		this.weatherData = weatherData;
		weatherData.registerObserver(this); // 透過建構子，將畫面註冊為觀察者
	}

	public void update(float temperature, float humidity, float pressure) {
		this.temperature = temperature;
		this.humidity = humidity;
		display(); // 在MVC的地方有更好的寫法
	}

	public void display() {
		System.out.println("Current conditions: " + temperature
			+ "F degrees and " + humidity + "% humidity");
	}
}
```

5. 寫一個測試項，確認訂閱會收到訊息，取消訂閱就不再收到訊息

```java
public class WeatherStation {

	public static void main(String[] args) {
		WeatherData weatherData = new WeatherData();

		CurrentConditionsDisplay currentDisplay =
			new CurrentConditionsDisplay(weatherData); // 透過建構子的方式，在方法裡面對Subject註冊自己

		weatherData.setMeasurements(80, 65, 30.4f); //數據更新，會收到update，並display
		weatherData.removeObserver(currentDisplay); //退出訂閱，不在收到通知
		weatherData.setMeasurements(62, 90, 28.1f); //數據更新，不會收到update
	}
}
```

## Pull Model / Pull Model

-   Push model
    -   推送所有資料給 Observer
    -   Subject 要知道 Observer 需要什麼，彈性較差
    -   Observer 會接收到不必要的資料
    -   好處是不需要保留 Subject 的引用

```java
public class CurrentConditionsDisplay implements Observer, DisplayElement {
	private float temperature;
	private float humidity;

	public void update(float temperature, float humidity, float pressure) { // Observer會接收到不必要的資料，例如pressure
		this.temperature = temperature;
		this.humidity = humidity;
		display();
	}

	public void display() {...}
}
```

-   Pull model
    -   提供必要的資料或其來源(如 data id 或 subject 本身)給 Observer，由 Observer 自行取得相對關資料
    -   每個 Observer 都要重新取得資料，效率較差
    -   壞處是需要保留 Subject 的引用

```java
// Subject
public void notifyObservers() {
    for (Observer observer : observers) {
        observer.update(); // 不傳遞參數
    }
}

// ----------------------------------------------------------------

// Object interface
public interface Observer {
	public void update(); // 沒有參數
}
// Object
public void update() {
    this.temperature = weatherData.getTemperature();
    this.humidity = weatherData.getHumidity();
    display();
}
```

## 觀察者模式(Observer Pattern) vs 發布/訂閱模式(Publish/Subscribe Pattern)

|   比較   |               觀察者模式(Observer Pattern)                |        發布/訂閱模式(Publish/Subscribe Pattern)        |
| :------: | :-------------------------------------------------------: | :----------------------------------------------------: |
| 模式類別 |                      Design Pattern                       |                   Messaging Pattern                    |
| 知道對方 | 觀察者知道 Subject 的，Subject 也一直保持對觀察者進行記錄 | 發布者和訂閱者不知道對方的存在，只透過消息代理進行通訊 |
|  耦合性  |                         相對耦合                          |                     組件是鬆散耦合                     |
|   同步   |                  大多是同步(synchronous)                  |           大多是異步(asynchronous)(消息對列)           |

### 發布/訂閱模式

![發布/訂閱模式](./%E7%99%BC%E4%BD%88%E8%A8%82%E9%96%B1%E6%A8%A1%E5%BC%8F.png?raw=true)

1. 模組的解偶

    > 發佈者(Publisher)和訂閱者(Subscriber)之間，透過中間人(broker)或 Message/Event Bus 來解偶
    > 就像訂閱某個粉專，訂閱者不需要知道發文的小編是誰

2. 時間的解偶
    > 發佈訊息時，訂閱者不一定在線上，採用先存再送(store-and-forward)的機制

## 觀察者模式 + 中介者模式

> [!WARNING]
> 一般的觀察者模式 Subject 仍需要保留 Observer 的引用，無法真正解偶

![觀察者模式 + 中介者模式](./%E8%A7%80%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8Fplus%E4%B8%AD%E4%BB%8B%E8%80%85%E6%A8%A1%E5%BC%8F.png)

1. **封裝複雜的更新語意**。當目標與觀察者的依賴關係特別複雜時，可能需要一個維護這些關係的對象，稱作**更改管理器(ChangeManager)**。
2. ChangeManager 是一個 Mediator(中介者)模式的實例，通常是一個 Singleleton(單例)

**ChangeManager 有三個職責：**

1. 將一個 Subject 映射到他的觀察者，並提供一個介面來維護這個映射，這就不用由 Subject 來維護對觀察者的引用
2. 定義一個特定的更新策略
3. 根據一個目標請求，更新所有依賴於這個目標的觀察者

![ChangeManager](./%E8%A7%80%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8Fplus%E4%B8%AD%E4%BB%8B%E8%80%85%E6%A8%A1%E5%BC%8F.gif)

-   當一個觀察者觀察多個目標時，DAGChangeManager 要更好用一些，他可以保證觀察者僅接受一個更新，而不會接受到多個冗余的更新。
-   當不存在重複更新時，SimpleChangeManager。
