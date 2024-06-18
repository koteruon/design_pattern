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

> [!TIP]
>
> **命令模式**可將請求封裝成物件，讓你可以將請求、佇列或紀錄等物件參數化，並支援可復原的的操作。

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
Light light = new Light("Living Room"); // 客廳電燈的物件
LightOnCommand lightOn = new LightOnCommand(light); // 客廳電燈的命令
Light light = new Light("Kitchen"); // 廚房電燈的物件
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
> NoCommand 是一個**null 物件**。null 物件很適合在你無法回傳有意義的物件，而且不想讓用戶端處理**沒有東西可用的(null)的情況**時使用。
>
> 你會發現很多設計模式都有 Null 物件，甚至有人將「Null Object」視為一種設計模式

### 文件撰寫

![遙控器設計API](./%E9%81%99%E6%8E%A7%E5%99%A8%E8%A8%AD%E8%A8%88API.png)

### 使用 lambda 優化

```java
// 原始的寫法
public class RemoteLoader {

    public static void main(String[] args) {
        RemoteControl remoteControl = new RemoteControl();

        Light livingRoomLight = new Light("Living Room");

        LightOnCommand livingRoomLightOn =
                new LightOnCommand(livingRoomLight); // 可以移除具體的Command物件
        LightOffCommand livingRoomLightOff =
                new LightOffCommand(livingRoomLight);

        remoteControl.setCommand(0, livingRoomLightOn, livingRoomLightOff);
    }
}

// 使用lambda寫法
public class RemoteLoader {

    public static void main(String[] args) {
        RemoteControl remoteControl = new RemoteControl();

        Light livingRoomLight = new Light("Living Room");

        remoteControl.setCommand(0, () -> livingRoomLight.on(),
                                    () -> livingRoomLight.off()); // 我們將具體command寫成lambda運算式
    }
}
```

> [!NOTE]
>
> 這種做法只能在 Command 介面只有**一個**抽象方法時使用，一旦加在第二個抽象方法，lambda 簡寫就無法使用

## 復原按鈕

### 電燈開關恢復

-   假設客廳的電燈是關閉的，當按下 on 之後，電燈會開啟。當你按下復原的時候，理論上電燈會關閉。

1. 讓 Command 介面可以支援復原，因此建立一個 execute()方法對應的 undo()方法

```java
public interface Command {
    public void execute();
    public void undo(); // 建立一個新的undo方法
}
```

2. 我們先處理簡單的 LightOnCommand，如果 On 就是上一次被呼叫的方法，我們知道 undo 必須呼叫 Off 方法，LightOffCommand 同理

```java
public class LightOnCommand implements Command {
    Light light;
    public LightOnCommand(Light light) {
        this.light = light;
    }

    public void execute() {
        light.on();
    }

    public void undo() { // execute 會將電燈開起，所以undo只要將電燈關閉即可
        light.off();
    }
}

public class LightOffCommand implements Command {
    Light light;
    public LightOffCommand(Light light) {
        this.light = light;
    }

    public void execute() {
        light.off();
    }

    public void undo() { // 同理undo將電燈開起
        light.on();
    }
}
```

3. 我們必須讓遙控器追蹤上一次被按下的按鈕以及 undo 按鈕。在 Remote Control 類別中，加入一個新的實例變數來記錄上一個呼叫的 command，當 undo 按下時，讀取那個 command，並呼叫它的 undo()方法

```java
public class RemoteControlWithUndo {
    Command[] onCommands;
    Command[] offCommands;
    Command undoCommand; // 將上一次執行的command存在這裡，讓undo按鈕使用

    public RemoteControlWithUndo() {
        onCommands = new Command[7];
        offCommands = new Command[7];

        Command noCommand = new NoCommand();
        for(int i=0;i<7;i++) {
            onCommands[i] = noCommand;
            offCommands[i] = noCommand;
        }
        undoCommand = noCommand; // undo與其他的位置一樣，在一開始被設定為NoCommand，所以在按下任何其他按鈕之前，按下undo不會做任何事情
    }

    public void setCommand(int slot, Command onCommand, Command offCommand) {
        onCommands[slot] = onCommand;
        offCommands[slot] = offCommand;
    }

    public void onButtonWasPushed(int slot) {
        onCommands[slot].execute();
        undoCommand = onCommands[slot]; // 將他的參考存入undoCommand之中
    }

    public void offButtonWasPushed(int slot) {
        offCommands[slot].execute();
        undoCommand = offCommands[slot]; // 將他的參考存入undoCommand之中
    }

    public void undoButtonWasPushed() { // 當undo按鈕被按下時，我們呼叫undoCommand所儲存的command的undo方法()，它會恢復上一次執行的command動作。
        undoCommand.undo();
    }

    public String toString() {
        StringBuffer stringBuff = new StringBuffer();
        stringBuff.append("\n------ Remote Control -------\n");
        for (int i = 0; i < onCommands.length; i++) {
            stringBuff.append("[slot " + i + "] " + onCommands[i].getClass().getName()
                + "    " + offCommands[i].getClass().getName() + "\n");
        }
        stringBuff.append("[undo] " + undoCommand.getClass().getName() + "\n"); // 加入undo的測試
        return stringBuff.toString();
    }
}
```

4. 測試電燈恢復

```java
public class RemoteLoader {

    public static void main(String[] args) {
        RemoteControlWithUndo remoteControl = new RemoteControlWithUndo();

        Light livingRoomLight = new Light("Living Room");

        LightOnCommand livingRoomLightOn =
                new LightOnCommand(livingRoomLight);
        LightOffCommand livingRoomLightOff =
                new LightOffCommand(livingRoomLight);

        remoteControl.setCommand(0, livingRoomLightOn, livingRoomLightOff);

        remoteControl.onButtonWasPushed(0); // Light is on
        remoteControl.offButtonWasPushed(0); // Light is off
        System.out.println(remoteControl); // [undo] LightOffCommand
        remoteControl.undoButtonWasPushed(); // Light is on
        remoteControl.offButtonWasPushed(0); // Light is off
        remoteControl.onButtonWasPushed(0); // Light is on
        System.out.println(remoteControl); // [undo] LightOnCommand
        remoteControl.undoButtonWasPushed(); // Light is off
    }
}
```

### 使用狀態來實作恢復

-   例如 CeilingFan 可以設定一些速度，也有 OFF 方法

1. 查看 CeilingFan 的原始碼

```java
public class CeilingFan {
    public static final int HIGH = 3;
    public static final int MEDIUM = 2;
    public static final int LOW = 1;
    public static final int OFF = 0;
    String location;
    int speed; // 類別保存一些區域狀態，代表吊扇的速度

    public CeilingFan(String location) {
        this.location = location;
        speed = OFF;
    }

    // 用這些方法設定吊扇的速度
    public void high() {
        speed = HIGH;
        System.out.println(location + " ceiling fan is on high");
    }

    public void medium() {
        speed = MEDIUM;
        System.out.println(location + " ceiling fan is on medium");
    }

    public void low() {
        speed = LOW;
        System.out.println(location + " ceiling fan is on low");
    }

    public void off() {
        speed = OFF;
        System.out.println(location + " ceiling fan is off");
    }

    // 可以使用這個方法取得目前吊扇的速度
    public int getSpeed() {
        return speed;
    }
}
```

2. 為吊扇的 command 加入恢復功能

```java
public class CeilingFanHighCommand implements Command { // High、Medium、Low、Off各需要一個command物件
    CeilingFan ceilingFan;
    int prevSpeed; // 加入區域變數來記錄吊扇上一次的速度

    public CeilingFanHighCommand(CeilingFan ceilingFan) {
        this.ceilingFan = ceilingFan;
    }

    public void execute() {
        prevSpeed = ceilingFan.getSpeed(); // 再更改吊扇速度前，先記錄他之前的狀態，在恢復的時候使用
        ceilingFan.high();
    }

    public void undo() { // 在恢復時，將吊扇的速度設回去他之前的速度
        if (prevSpeed == CeilingFan.HIGH) {
            ceilingFan.high();
        } else if (prevSpeed == CeilingFan.MEDIUM) {
            ceilingFan.medium();
        } else if (prevSpeed == CeilingFan.LOW) {
            ceilingFan.low();
        } else if (prevSpeed == CeilingFan.OFF) {
            ceilingFan.off();
        }
    }
}
```

3. 測試吊扇

```java

package headfirst.designpatterns.command.undo;

public class RemoteLoader {

    public static void main(String[] args) {
        RemoteControlWithUndo remoteControl = new RemoteControlWithUndo();

        CeilingFan ceilingFan = new CeilingFan("Living Room");

        // 這裡實例化三個command
        CeilingFanMediumCommand ceilingFanMedium =
                new CeilingFanMediumCommand(ceilingFan);
        CeilingFanHighCommand ceilingFanHigh =
                new CeilingFanHighCommand(ceilingFan);
        CeilingFanOffCommand ceilingFanOff =
                new CeilingFanOffCommand(ceilingFan);

        // 將medium放入第0個位置，將high放入第1個位置，並且也載入off command
        remoteControl.setCommand(0, ceilingFanMedium, ceilingFanOff);
        remoteControl.setCommand(1, ceilingFanHigh, ceilingFanOff);

        remoteControl.onButtonWasPushed(0); // Living Room ceiling fan is on medium
        remoteControl.offButtonWasPushed(0); // Living Room ceiling fan is off
        System.out.println(remoteControl); // [undo] CeilingFanOffCommand
        remoteControl.undoButtonWasPushed(); // Living Room ceiling fan is on medium

        remoteControl.onButtonWasPushed(1); // Living Room ceiling fan is on high
        System.out.println(remoteControl); // [undo] CeilingFanHighCommand
        remoteControl.undoButtonWasPushed(); // Living Room ceiling fan is on medium
    }
}
```

## 使用巨集 Command(派對模式)

### 需求

-   希望遙控器可以用一顆按鈕同時調暗燈光、打開音響和電視、讓熱水浴缸開始加溫

### 創建巨集的Command

1. 製作一種新的 Command，讓他可以執行其他多個Command

```java
public class MacroCommand implements Command {
    Command[] commands;

    public MacroCommand(Command[] commands) { // 接收一個Command陣列，並存入MacroCommand
        this.commands = commands;
    }

    public void execute() {
        for (int i = 0; i < commands.length; i++) { // 當遙控器執行巨集時，一次執行這些command
            commands[i].execute();
        }
    }

    /**
     * NOTE:  these commands have to be done backwards to ensure
     * proper undo functionality
     */
    public void undo() {
        for (int i = commands.length -1; i >= 0; i--) {
            commands[i].undo();
        }
    }
}
```

2. 使用巨集Command

```java
public class RemoteLoader {

    public static void main(String[] args) {

        RemoteControl remoteControl = new RemoteControl();

        // 建立所有的設備：電燈、電視、音響、熱水浴缸
        Light light = new Light("Living Room");
        TV tv = new TV("Living Room");
        Stereo stereo = new Stereo("Living Room");
        Hottub hottub = new Hottub();

        // 建立所有的ON、OFF Command來控制他們
        LightOnCommand lightOn = new LightOnCommand(light);
        StereoOnCommand stereoOn = new StereoOnCommand(stereo);
        TVOnCommand tvOn = new TVOnCommand(tv);
        HottubOnCommand hottubOn = new HottubOnCommand(hottub);
        LightOffCommand lightOff = new LightOffCommand(light);
        StereoOffCommand stereoOff = new StereoOffCommand(stereo);
        TVOffCommand tvOff = new TVOffCommand(tv);
        HottubOffCommand hottubOff = new HottubOffCommand(hottub);

        // 建立On Command的陣列、Off Command的陣列
        Command[] partyOn = { lightOn, stereoOn, tvOn, hottubOn};
        Command[] partyOff = { lightOff, stereoOff, tvOff, hottubOff};

        // 並建立兩個對應的巨集來保存他們
        MacroCommand partyOnMacro = new MacroCommand(partyOn);
        MacroCommand partyOffMacro = new MacroCommand(partyOff);

        // 將巨集command指派給按鈕，和處理任何command一樣
        remoteControl.setCommand(0, partyOnMacro, partyOffMacro);

        System.out.println(remoteControl);
        System.out.println("--- Pushing Macro On---");
        remoteControl.onButtonWasPushed(0);
        /*
            Light is on
            Living Room stereo is on
            Living Room TV is on
            Living Room TV channel is set for DVD
            Hottub is heating to a steaming 104 degrees
            Hottub is bubbling!
        */
        System.out.println("--- Pushing Macro Off---");
        remoteControl.offButtonWasPushed(0);
        /*
            Light is off
            Living Room stereo is off
            Living Room TV is off
            Hottub is cooling to 98 degrees
        */
    }
}
```

