---
sidebar_position: 40
---

# Timers

With timers in SmartBeans you can schedule actions to be executed _after_ a specific time.

:::note
If you want to trigger actions _at_ specific times or on a regular schedule (similar to cron jobs), use the [time trigger](../reference/trigger/time) 
feature of Home Assistant. The SmartBeans timer is not designed for this kind of scheduled operations. Instead, it is
better suited for delaying actions after specific events or for implementing timeout mechanisms within automation flows.
:::

A timer in SmartBeans is an instance of the `Timer` class that can be instantiated programmatically by invoking the 
`createTimer()` method on the `SmartBeans` API. For example, consider a scenario where you have implemented a bean for
motion-controlled lighting: instead of turning off the lights immediately when no motion is detected, you might want to
implement a 30-second delay. This can be efficiently achieved using a timer.

````java
public class KitchenMotionControl implements SmartBean {
  
  private SmartBeans sb;

  @Entity("light.kitchen_ceiling")
  private Light ceilingLight;

  @OnStateTrigger(entity = "binary_sensor.motion_kitchen", to = "off")
  public void onNoMotionDetected() {
    Timer lightOffTimer = sb.createTimer(timer -> timer
        .setInterval(Duration.ofSeconds(30))
        .onElapsed(evt -> ceilingLight.turnOff())
    );
    lightOffTimer.start();
  }
}
````

## Using annotations

Alternatively, you can define a timer using the `@TimerDef` annotation. To implement this approach, create a field of 
type `Timer` in your SmartBean and annotate it with `@TimerDef`. The annotation allows you to specify various timer 
properties such as interval, repetition settings, and autostart behavior. SmartBeans will automatically instantiate 
the timer and inject it into this field.

A significant advantage of this method is the ability to use additional annotations on your SmartBean methods to connect
them directly to the timer. This annotation-driven approach results in more concise and readable code.

````java
public class KitchenMotionControl implements SmartBean {

  @TimerDef(interval = @Interval(seconds = 30))
  private Timer lightOffTimer;

  @Entity("light.kitchen_ceiling")
  private Light ceilingLight;

  @OnStateTrigger(entity = "binary_sensor.motion_kitchen", to = "off")
  public void onNoMotionDetected() {
    lightOffTimer.start();
  }
  
  @OnTimerElapsed("lightOffTimer")
  public void turnOffLight() {
    ceilingLight.turnOff();
  }
}
````

Another advantage of having the `Timer` object as a field of your bean is that you can invoke other operations on the timer 
as described below.

## Cancelling a timer

You can cancel a timer by invoking the `cancel()` method on the timer object. This immediately stops the timer and 
triggers a cancellation event. For example, in our motion detection scenario, if motion is detected again while the 
timer is running, you might want to prevent the light from turning off. In this case, you would cancel the timer,
ensuring that the light doesn't turn off after the 30-second delay.

````java
public class KitchenMotionControl implements SmartBean {

  @TimerDef(interval = @Interval(seconds = 30))
  private Timer lightOffTimer;

  @Entity("light.kitchen_ceiling")
  private Light ceilingLight;

  @OnStateTrigger(entity = "binary_sensor.motion_kitchen", to = "on")
  public void onMotionDetected() {
    if(lightOffTimer.isRunning()) {
      lightOffTimer.cancel();
    }
    ceilingLight.turnOn();
  }

  @OnStateTrigger(entity = "binary_sensor.motion_kitchen", to = "off")
  public void onNoMotionDetected() {
    lightOffTimer.start();
  }

  @OnTimerElapsed("lightOffTimer")
  public void turnOffLight() {
    ceilingLight.turnOff();
  }
}
````

## Repeating and autostart

SmartBeans timers can be configured to execute repeatedly at specified intervals. When a timer is initiated, the `elapsed`
event is triggered at each interval for the designated number of repetitions. If the timer is canceled during execution, 
all subsequent event firings are immediately terminated, and no further `elapsed` events will be triggered.

This timer will fire an `elapsed` event every 10 seconds for three repetitions.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  @TimerDef(interval = @Interval(seconds = 10), repeatCount = 3)
  private Timer timer;

  @OnTimerElapsed("timer")
  public void onElapsed(TimerEvent evt) {
    sb.log("Timer elapsed, there are %d repititions left.", evt.getRemainingRepeats());
  }

  @OnTimerCancelled("timer")
  public void onTimerCanceled(TimerEvent evt) {
    sb.log("Timer canceled with %d repititions left.", evt.getRemainingRepeats());
  }
}
````

The autostart feature enables a timer to initialize and begin execution immediately upon creation, eliminating the need 
for an explicit `start()` method invocation. For timers configured via annotations, execution starts automatically 
during bean deployment in SmartBeans.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  @TimerDef(interval = @Interval(seconds = 20), autostart = true)
  private Timer timer;

  @OnTimerElapsed("timer")
  public void onElapsed(TimerEvent evt) {
    sb.log("Bean is deplyed for %d seconds now.", evt.getInterval().toSeconds());
  }
}
````
