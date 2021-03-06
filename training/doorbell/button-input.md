# Implement a Doorbell Button

##实现一个门铃按钮

## This lesson teaches you to

##本节课你将学到

1.  Connect hardware to GPIO

2.  将硬件连接到 GPIO
3.  Initialize a peripheral driver


2. 初始化一个外部设备驱动
3. Manage peripheral connections


3. 管理外部设备的连接

## Try it out

##试试看

*   [Doorbell sample app](https://github.com/androidthings/doorbell)
*   [门铃示例应用](https://github.com/androidthings/doorbell)

A doorbell isn't much to speak of without a button for visitors to press and announce their presence at the door.

一个没有设置可供访客按下并宣告有客来访按钮的门是没有意义的。

In this lesson, you will learn to connect a pushbutton to a GPIO input, initialize it using Peripheral I/O, and listen for state changes on that input.

通过这节课，你将学会将一个开关连接到 GPIO 的输入端口，使用外设 I/O API 初始化它，并且监听输入时的状态变化。

## Connect the button

##连接开关

* * *

Using a breadboard, connect a pushbutton switch to the appropriate GPIO port on your board. To connect the button to your board:

将电路板上的按键开关连接到开发板中合适的GPIO接口上，为了将开关连接到开发板，你需要做如下一些事情：

1.  Connect one side of the button to the chosen GPIO input pin, and the other side to ground.

2.  将开关的一端连接到你选择的 GPIO 的输入引脚，另一个端接地。


2. Connect the same GPIO input pin to +3.3V through a pull-up resistor.


2. 用一个上拉电阻将这个 GPIO 引脚连接到 +3.3V 的电源上。

For this lesson, the following GPIO pins are assumed on each board:

在本节中，我们假定每个开发板都有 GPIO 引脚：

| Board                                    | Input Signal |
| ---------------------------------------- | ------------ |
| [NXP Pico i.MX7D](https://developer.android.google.cn/things/hardware/imx7d-pico-io.html) | GPIO_39      |
| [Raspberry Pi](https://developer.android.google.cn/things/hardware/raspberrypi-io.html) | BCM21        |

![""](https://developer.android.google.cn/things/images/doorbell-button-wiring.png)

> **Note:** See [Hardware 101](https://developer.android.google.cn/things/hardware/hardware-101.html) for more detail on connecting input and output components.

> **注意:** 查看  [Hardware 101](https://developer.android.google.cn/things/hardware/hardware-101.html)  了解连接关于输入/输出元器件的一些更详细的信息。

## Capture button press events

##捕获按钮按下事件

* * *

The `Button` peripheral driver handles the low-level logic of listening for GPIO state changes. It also debounces the input port by default, to avoid multiple trigger events from a single button press. To add the button driver to your app:

`Button` 外设驱动处理监听 GPIO 状态更改的底层逻辑。为了避免按下按钮一次触发多次事件，驱动会将输入口状态重置至默认值。你需要做如下一些事来添加按键驱动到你的应用中：

1.  Add a button peripheral to your app project by connecting it to an available GPIO port.

2.  通过将按钮连接到一个可用的 GPIO 接口来为你的工程添加一个按键外设。


2. Add the button driver dependency to your app-level `build.gradle` file:


2. 添加按钮驱动依赖到你的应用下的`build.gradle`文件中：

```groovy
  dependencies {
    ...

    compile 'com.google.android.things.contrib:driver-button:0.3'
}
```

3. Initialize a `ButtonInputDriver` with the connected GPIO port name, and the appropriate logic state indicating when the button is physically pressed.


3. 利用已连接的 GPIO 的端口名来初始化为一个 `ButtonInputDriver`，并根据按钮的按压状态显示对应的逻辑状态。
4. By using a `ButtonInputDriver` instead of a `Button`, you are associating Android key press/release events with the button connected to the GPIO. Specify which `KeyEvent` key code will be generated in the `ButtonInputDriver` constructor.


4. 使用 `ButtonInputDriver`  代替 `Button`，你可以将 Andoird 按键按下/释放事件和连接在 GPIO 的按钮相关联。同时在 `ButtonInputDriver` 的构造函数中指定指定触发特定 `keyEvent` 键码值。


5. Capture the key up or down events, as if they were being generated by a keyboard, by overriding the `Activity.onKeyUp` or `onKeyDown` method:


5. 通过重写 `Activity.onKeyUp` 或者 `onKeyDown` 方法来捕捉按钮按下或释放事件，就像他们是用键盘产生的事件一样：

   ```java
   public class DoorbellActivity extends Activity {

       /*
        * Driver for the doorbell button;
        */
       private ButtonInputDriver mButton;

       /**
        * The GPIO pin to activate for button presses.
        */
       private final String BUTTON_GPIO_PIN = ...;

       @Override
       public void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           Log.d(TAG, "Doorbell Activity created.");

           ...

           // Initialize the doorbell button driver
           try {
               mButton = new ButtonInputDriver(BUTTON_GPIO_PIN,
                   Button.LogicState.PRESSED_WHEN_LOW, KeyEvent.KEYCODE_ENTER); // The keycode to send
           } catch (IOException e) {
               Log.e(TAG, "button driver error", e);
           }
       }

       /**
        * Override key event callbacks.
        */
       @Override
       public boolean onKeyUp(int keyCode, KeyEvent event) {
           if (keyCode == KeyEvent.KEYCODE_ENTER) {
               // Doorbell rang!
               Log.d(TAG, "button pressed");
               return true;
           }
           return super.onKeyUp(keyCode, event);
       }
   }
   ```

## Manage the connection

## 管理连接

* * *

You should manage the peripheral connection according to the application component lifecycle. You already initialized the `ButtonInputDriver` component when the parent Activity was created. Now close the connection when the Activity is stopped or destroyed.

你应该根据应用程序组件的生命周期来管理外设的连接。在 Activity 被创建的时候你已经初始化了 `ButtonInputDriver` 。当 Activity 被停止使用或者被销毁时关闭连接。

```java
public class DoorbellActivity extends Activity {
    ...

    private ButtonInputDriver mButton;

    @Override
    protected void onDestroy() {
        super.onDestroy();
        ...

        try {
            mButton.close();
        } catch (IOException e) {
            Log.e(TAG, "button driver error", e);
        }
    }
}
```

