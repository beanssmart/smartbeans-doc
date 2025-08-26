---
sidebar_position: 70
---

# Bean state

Like any other Java class, a bean can maintain state through member variables. However, this state is not persistent 
across bean redeployments, server restarts, or when SmartBeans loses connection to Home Assistant. 

Consider our motion-controlled light bean example: there may be scenarios where the light is already on when motion is 
detected. In such cases, we don't want to turn the light off when motion is no longer detected. Therefore, we need to
store the light's initial state before activating it via motion detection, and then check this stored state when 
determining the appropriate action after motion ceases.

````java
public class KitchenMotionControl implements SmartBean {
  
  @Entity("light.kitchen_ceiling")
  private Light light;
  
  private boolean alreadyOn;
  
  @OnStateTrigger(entity = "binary_sensor.kitchen_motion", to = "on")
  public void onMotion() {
    alreadyOn = light.isOn();
    light.turnOn();
  }
  
  @OnStateTrigger(entity = "binary_sensor.kitchen_motion", to = "off")
  public void onNoMotion() {
    if(!alreadyOn) {
      light.turnOff();
    }
  }
}
````

This example works like intended most of the time, but the state of `alreadyOn` is lost when the bean is recreated. As
a result, if motion is detected while the light is already on, and then the server restarts, the light will be 
incorrectly turned off when motion is no longer detected.

SmartBeans offers a straightforward solution through the `@State` annotation. By applying this annotation to specific 
member variables, you can ensure their persistence across redeployment cycles. During undeployment, SmartBeans 
automatically serializes all fields marked with the `@State` annotation into a JSON file. Upon subsequent redeployment,
the system automatically deserializes this JSON data to restore the bean's state to its previous condition, ensuring
continuity of operation without data loss.

````java
public class KitchenMotionControl implements SmartBean {

  @Entity("light.kitchen_ceiling")
  private Light light;

  @State
  private boolean alreadyOn;
}
````

As a side effect this state is populated as an attribute of the [bean entity](devices) in Home Assistant. This attribute
can be visualized in dashboards and its historical values can be tracked through the built-in history functionality.

## Datatypes

SmartBeans supports the following data types for `@State` fields:
- `boolean`/`Boolean`
- `int`/`Integer`
- `double`/`Double`
- `String`
- `Instant`
- Any `Enum` type
- Any class implementing the `HasState` interface
- `List<T>` where `T` is any of the supported types above

Classes that implement the `HasState` interface can manage complex state data. Any field within such a class can be
annotated with `@State`, which marks it for state serialization. When a field annotated with `@State` contains an object
reference, the state of that referenced object will be serialized as a nested structure within the parent bean's JSON
state representation.

Have a look at this bean:

````java
@SmartBeanDef(beanDevice = @BeanDevice(type = BeanDevice.EntityType.BINARY_SENSOR))
public class ASampleBean implements SmartBean {

  @State
  private String foo = "bar";
  
  @State
  private Color color = new Color();
  
  public class Color implements HasState {
    @State public int red = 123;
    @State public int green = 54;
    @State public int blue = 213;
  }
}
````

The corresponding JSON state representation will look like this:

````json
{
  "foo": "bar",
  "color": {
    "red": 123,
    "green": 54,
    "blue": 213
  }
}
````

And the attributes of the created entity in Home Assistant will look like this:

````yaml
color:
  red: 123
  green: 54
  blue: 213
foo: bar
````

If you prefer to avoid the nested structure, you can utilize the `nested` attribute of the `@State` annotation to switch 
to `FLAT` mode. When this mode is enabled, the state of the annotated field will be serialized as individual 
attributes at the root level rather than as a hierarchical object.

```java
@SmartBeanDef(beanDevice = @BeanDevice(type = BeanDevice.EntityType.BINARY_SENSOR))
public class ASampleBean implements SmartBean {

  @State
  private String foo = "bar";

  @State(nested = NestedMode.FLAT)
  private Color color = new Color();
}
```

Will result in the following attributes in Home Assistant:

````yaml
color.blue: 213
color.red: 123
color.green: 54
foo: bar
````

This can be easier to work with in some dashboard components.

## Read/write state

In certain edge cases, annotation-based state persistence may not be suitable. For instance, when you need to persist 
the state of an unsupported data type, such as a field of type `LocalDate`, you can leverage the `writeState` and 
`readState` methods provided by the `HasState` interface. Since all SmartBean implementations inherit from this 
interface, you can utilize these methods to explicitly control the persistence and restoration of your bean's state.

````java
@SmartBeanDef(beanDevice = @BeanDevice(type = BeanDevice.EntityType.BINARY_SENSOR))
public class ASampleBean implements SmartBean {

  private LocalDate someDate;

  @Override
  public void writeState(StateWriter writer) {
    writer.setString("someDate", someDate.toString());
  }

  @Override
  public void readState(StateReader reader) {
    someDate = reader.getString("someDate")
        .map(LocalDate::parse)
        .orElseGet(LocalDate::now);
  }
}
````

:::note
Please note that `readState()` is also invoked during the initial bean deployment, which means any state being read may 
return `null`. The `readState()` method implementation must handle this `null` case gracefully to ensure proper initialization.
:::

## Timers

Certain objects in SmartBeans already implement the `HasState` interface, allowing them to function as bean state
components. The [Timer](timer) class is a prime example of this implementation. Any field of type `Timer` can be 
annotated with `@State`, enabling its state to be serialized and restored as an integral part of the bean's state 
persistence mechanism. As a result, a running timer maintains continuity across system redeployments. This ensures that
timer-triggered events will execute as scheduled even if the SmartBeans server experiences a restart during the timer's
active period. If the timer elapses while the server is offline, the elapsed event will be triggered immediately upon
server restart - more specifically, during bean deployment.

````java
@SmartBeanDef(beanDevice = @BeanDevice(type = BeanDevice.EntityType.SWITCH))
public class KitchenMotionControl implements SmartBean {
  
  @Entity("light.kitchen_ceiling")
  private Light light;
  
  @TimerDef(interval = @Interval(seconds = 60))
  @State
  private Timer lightOffTimer;
  
  @OnStateTrigger(entity = "binary_sensor.kitchen_motion", to = "off")
  public void onNoMotion() {
    lightOffTimer.start();
  }
  
  @OnTimerElapsed("lightOffTimer")
  public void turnOffLight() {
    light.turnOff();
  }
}
````