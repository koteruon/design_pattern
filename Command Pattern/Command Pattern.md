# Command Pattern (命令模式) (Action) (Transaction)

> **命令模式**是一種行為設計模式，可將請求封裝成物件，讓你可以將請求、佇列或紀錄等物件參數化，並支援可復原的的操作

### 命令模式結構

![命令模式結構](./%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F%E6%9E%B6%E6%A7%8B.png)

### 命令模式的應用場景

1. **如果需要把動作變成參數化對象**
    > 命令模式可將特定的方法轉化成獨立對象。你可以將命令作為方法的參數進行傳遞，將命令保存在其他對象中，或是在執行期切換已連接的命令。
2. **如果想要將動作放入隊列中、動作的執行或遠程執行動作**
    > 同其他對象一樣，命令也可以實現序列化，從而方便的寫入文件或數據庫中。一段時間後，再將他恢復成最初的命令對象。因此，可以延遲或計劃命令的執行。也可以將命令放入隊列、紀錄命令、通過網路傳送命令。
3. **如果你想要實現操作回滾功能**
    > 盡管有很多方法可以實現撤銷和恢復功能，但命令模式是其中常用的一種。
    > 為了能夠回滾操作，你需要實現已執行操作的歷史紀錄功能。對所有已執行命令對象及其相關程序狀態備份的 stack 結構，但是這種方法有兩個缺點。
    >
    > 1. 程式狀態的保存功能並不容易實現，因為部分狀態為私有。可以使用**備忘錄模式**來在一定程度上解決這個問題。
    >
    > 2. 備份狀態可能會占用大量內存。有時需要借助另一種實現方式：反向操作。反向操作也有代價：他可能會很難甚至無法實現。

### 優缺點

:o:**優點**

1. 單一職責原則。可以解藕觸發和執行操作的類。
2. 開放封閉原則。可以在不修改已有客戶端程式碼的情況下，在程式碼中創建先的命令。
3. 可以實現撤銷和恢復功能
4. 可以實現操作的延遲執行
5. 可以將一組簡單命令組合成一個複雜命令

:x:**缺點**

1. 程式碼會變得更加複雜，因為你在發送者和接收者之間增加了一個全新的層次。

## 命令模式(Command Pattern)

command 物件藉著將準備送給 receiver 的一組行動綁在一起，來**封裝**請求。command 物件將動作和 receiver 都包在它裡面，只公開一個方法 excute()。當你呼叫 execute()時，就會呼叫 receiver 的動作。在外面，其他的物件都不知道哪個 receiver 執行什麼動作，他們只知道呼叫 execute()方法之後，他們的請求就會被處理。

### 遇到的需求

-   我們想要設計一個控制器，上面可以登入很多設備，另外有 ON 和 OFF 的按鈕來控制設備，以及最後一個 UNDO 可以復原的按鈕

![控制器](./%E6%8E%A7%E5%88%B6%E5%99%A8.png)

### 遇到的問題

-   廠商的類別有非常多，而且有著很不一樣的介面，因此希望遙控器不需要知道太多家電的介面細節，也不想在遙控器裡面有一堆家電的 if，(如： if slot1 == Light, then light.on())

![廠商的類別](./%E5%BB%A0%E5%95%86%E7%9A%84%E9%A1%9E%E5%88%A5.png)

### 命令物件

-   命令物件可以用特定的物件(例如客廳電燈物件)來封裝做某件事情的請求(例如打開電燈)
-   我們幫遙控器的按鈕指定一個命令物件，按下按鈕時，只需要呼叫命令物件做某項工作即可

1. 實作 Command 介面，所有的 Command 物件都實作同一個介面，介面有一個通用方法(execute)

```java
public interface Command {
	public void execute(); // 只需要一個execute方法
}
```

2. 實作一個 Command 來開燈。(或是說將廠商的物件封裝在 Command 介面中)

```java
public class LightOnCommand implements Command { // 我們要實作Command介面
	Light light; // 家電的物件

	public LightOnCommand(Light light) { // 讓建構式接收這個command要控制的電燈，假設是客廳電燈，並將它存入變數中。當execute被呼叫的時候，接收請求的就是這個物件。
		this.light = light;
	}

	public void execute() { // execute()方法呼叫物件的on()方法
		light.on();
	}
}
```

3. 建立 Invoker，使用 command 物件。假設我們的遙控器只有一個按鈕及其對應的位置

```java
// This is the invoker
public class SimpleRemoteControl {
	Command slot;

	public SimpleRemoteControl() {}

	public void setCommand(Command command) {
		slot = command;
	}

	public void buttonWasPressed() {
		slot.execute();
	}
}
```

4. 建立簡單的測試程式來使用遙控器

```java
public class RemoteControlTest { // 這是命令模式中的client
	public static void main(String[] args) {
		SimpleRemoteControl remote = new SimpleRemoteControl(); // 遙控器是一個invoker，它會收到一個command物件，那個command物件可以用來發出請求
		Light light = new Light(); // 建立Light物件，它是接受請求的Receiver
		LightOnCommand lightOn = new LightOnCommand(light); // 在這裡建立一個command，並將Receiver傳給他

		remote.setCommand(lightOn); // 將commnad傳給Invoker
		remote.buttonWasPressed(); // 模擬按鈕被按下
    }
}
```

### 完整的遙控器製作

1. 再遙控器中有七個位置，每一個位置都有開、關按鈕，可以將 command 指派給遙控器

```java
onCommands[0] = onCommand;
offCommands[0] = offCommand;
```

2. 當我們建立將被載入遙控器的 Command 時，我們會建立*一個客廳電燈物件的 LightCommand*，另外建立*一個廚房電燈物件的 LightCommand*。因此**請求的 receiver 與封裝它的 command 是綁定的**。所以，當按鈕被按下時，**沒有人在乎電燈是哪一個**，當 execute()方法被呼叫時，對的事情就會發生。

```java
Light light = new Light(); // 客廳電燈的物件
LightOnCommand lightOn = new LightOnCommand(light); // 客廳電燈的命令
Light light = new Light(); // 廚房電燈的物件
LightOnCommand lightOn = new LightOnCommand(light); // 廚房電燈的命令
```

3. 將 Command 指派給位置

![完成的控制器](./%E5%AE%8C%E6%88%90%E7%9A%84%E6%8E%A7%E5%88%B6%E5%99%A8.png)

```java
// This is the invoker
public class RemoteControl { // 這一次控制器會處理7個On和Off command，我們將用相應的陣列來保存他們
	Command[] onCommands;
	Command[] offCommands;

	public RemoteControl() { // 建構式裡，我們只需要實例化與初始化On 和 Off 陣列即可。
		onCommands = new Command[7];
		offCommands = new Command[7];

		Command noCommand = new NoCommand();
		for (int i = 0; i < 7; i++) {
			onCommands[i] = noCommand;
			offCommands[i] = noCommand;
		}
	}

	public void setCommand(int slot, Command onCommand, Command offCommand) { // 接收位置，以及要存入那個位置的 On 和 Off command
		onCommands[slot] = onCommand; // 將這些command放入On與Off陣列，以備後用
		offCommands[slot] = offCommand;
	}

	public void onButtonWasPushed(int slot) { // 當On 或 Off 按鈕被按下時，由硬體負責呼叫對應的方法
		onCommands[slot].execute();
	}

	public void offButtonWasPushed(int slot) {
		offCommands[slot].execute();
	}

	public String toString() { // 列印出每一個位置及其command，測試時使用到它
		StringBuffer stringBuff = new StringBuffer();
		stringBuff.append("\n------ Remote Control -------\n");
		for (int i = 0; i < onCommands.length; i++) {
			stringBuff.append("[slot " + i + "] " + onCommands[i].getClass().getName()
				+ "    " + offCommands[i].getClass().getName() + "\n");
		}
		return stringBuff.toString();
	}
}
```

4. 實作 Command，使用上面寫好的 LightOnCommand，以及為 Stereo 編寫 On command

```java
// Light on command
public class LightOnCommand implements Command {
	Light light;

	public LightOnCommand(Light light) {
		this.light = light;
	}

	public void execute() {
		light.on();
	}
}

// stereo On with CD command
public class StereoOnWithCDCommand implements Command {
	Stereo stereo;

	public StereoOnWithCDCommand(Stereo stereo) {
		this.stereo = stereo; /// 我們接收要控制的stereo實例，並將它存入實例變數
	}

	public void execute() {
		stereo.on(); // 將它打開
		stereo.setCD(); // 設定播放CD
		stereo.setVolume(11); // 將音量設成11
	}
}
```

5. 逐步測試遙控器

```java
public class RemoteLoader {

	public static void main(String[] args) {
		RemoteControl remoteControl = new RemoteControl();

        // 在合適的位置建立所有的設備
		Light livingRoomLight = new Light("Living Room");
		Light kitchenLight = new Light("Kitchen");
		CeilingFan ceilingFan= new CeilingFan("Living Room");
		GarageDoor garageDoor = new GarageDoor("Garage");
		Stereo stereo = new Stereo("Living Room");

        // 建立所有的Light Command
		LightOnCommand livingRoomLightOn =
				new LightOnCommand(livingRoomLight);
		LightOffCommand livingRoomLightOff =
				new LightOffCommand(livingRoomLight);
		LightOnCommand kitchenLightOn =
				new LightOnCommand(kitchenLight);
		LightOffCommand kitchenLightOff =
				new LightOffCommand(kitchenLight);

        // 建立吊扇的On與Off Command
		CeilingFanOnCommand ceilingFanOn =
				new CeilingFanOnCommand(ceilingFan);
		CeilingFanOffCommand ceilingFanOff =
				new CeilingFanOffCommand(ceilingFan);

        // 建立車庫的Up與Down Command
		GarageDoorUpCommand garageDoorUp =
				new GarageDoorUpCommand(garageDoor);
		GarageDoorDownCommand garageDoorDown =
				new GarageDoorDownCommand(garageDoor);

        // 建立音響的On與Off Command
		StereoOnWithCDCommand stereoOnWithCD =
				new StereoOnWithCDCommand(stereo);
		StereoOffCommand  stereoOff =
				new StereoOffCommand(stereo);

        // 完成所有command之後，將他們載入遙控器的位置
		remoteControl.setCommand(0, livingRoomLightOn, livingRoomLightOff);
		remoteControl.setCommand(1, kitchenLightOn, kitchenLightOff);
		remoteControl.setCommand(2, ceilingFanOn, ceilingFanOff);
		remoteControl.setCommand(3, stereoOnWithCD, stereoOff);

        // 利用toString()方法，印出每一個遙控器位置，以及指派給它的command
		System.out.println(remoteControl);

        // 逐步按下每一個位置的on和off
		remoteControl.onButtonWasPushed(0);
		remoteControl.offButtonWasPushed(0);
		remoteControl.onButtonWasPushed(1);
		remoteControl.offButtonWasPushed(1);
		remoteControl.onButtonWasPushed(2);
		remoteControl.offButtonWasPushed(2);
		remoteControl.onButtonWasPushed(3);
		remoteControl.offButtonWasPushed(3);
	}
}
```

6. 建立 NoCommand 以取代 if 判斷

-   為了避免每次引用位置都要檢查它有沒有載入 command

```java
public void onButtonWasPushed(int slot) {
    if (onCommands[slot] != null) { // 看起來很醜
        onCommands[slot].execute();
    }
}
```

-   創建一個不做任何事情的 command

```java
public class NoCommand implements Command {
	public void execute() { }
}

```

-   在上面例子中 RemoteControl 的建構式中，將 NoCommand 預先指派給每一個位置，每一個位置就一定有一個 command 可以呼叫

```java
Command noCommand = new NoCommand();
for (int i = 0; i < 7; i++) {
    onCommands[i] = noCommand;
    offCommands[i] = noCommand;
}
```

> [!NOTE]
>
> NoCommand 是一個**null 物件**。null 物件很適合在你無法回傳有意義的物件，而且不想讓用戶端處理**沒有東西可用的(null)**的情況時使用。
>
> 你會發現很多設計模式都有Null物件，甚至有人將「Null Object」視為一種設計模式

### 使用 lambda 優化
