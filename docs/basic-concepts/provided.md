---
sidebar_position: 80
---

# Provided Entities

A SmartBean can provide entities to Home Assistant. These are additional entities that are created through the MQTT 
discovery mechanism and are added to the [bean device](devices) in Home Assistant. These entities can be used to
populate data or to trigger actions of the bean from Home Assistant.

## Populate Data

To expose data from a bean to Home Assistant, you can define one or more sensors. For example, if a motion-controlled 
light bean needs to provide a binary sensor indicating whether the light was activated by motion, you can declare a
field of type `ProvidedBinarySensor` in the bean and annotate it with `@Provided`. SmartBeans will then automatically
register the corresponding binary sensor in Home Assistant and inject a `ProvidedBinarySensor` instance into the bean.
You can update the sensor state by invoking the `setState()` method on the injected object.

Use the properties of the `@Provided` annotation to configure the entity created in Home Assistant, such as setting its
`entity_id`, friendly name, icon, and other attributes.

````java
public class KitchenMotionControl implements SmartBean {
  
  @Provided(
      entityId = "binary_sensor.kitchen_light_by_motion",
      icon = "mdi:light_bulb",
      friendlyName = "Kitchen light by motion"
  )
  private ProvidedBinarySensor lightByMotion;

  @OnStateTrigger(entity = "binary_sensor.kitchen_motion", to = "on")
  public void onMotion() {
    lightByMotion.setState(BinarySensor.State.ON);
  }
  
  @OnTimerElapsed("lightOffTimer")
  public void turnOffLight() {
    lightByMotion.setState(BinarySensor.State.OFF);
  }
}
````

## Trigger Actions

You can also trigger bean actions through provided entities. SmartBeans allows you to create a button in Home Assistant 
that links directly to a method in your bean. When the button is pressed, SmartBeans invokes the designated method. 

To implement this functionality create a field of type `ProvidedButton` in your bean and Annotate it with `@Provided`.
SmartBeans will then automatically register the button in Home Assistant and inject a `ProvidedButton` instance into 
your bean

You have two options to handle button press events:
1. Use the `onPressed()` method of the injected object to register a listener for button presses
2. Alternatively, use the `@OnButtonPressed` annotation on a method in your bean, as demonstrated in the example below

````java
@SmartBeanDef(beanDevice = @BeanDevice(type = BeanDevice.EntityType.BINARY_SENSOR))
public class ASampleBean implements SmartBean {
  
  private SmartBeans sb;

  @Provided(
      entityId = "button.manual_bean_action",
      icon = "mdi:button-cursor",
      friendlyName = "Manual Action"
  )
  private ProvidedButton manualAction;

  @OnButtonPressed("manualAction")
  public void manualButtonTriggered() {
    sb.log("Manual action triggered");
  }
}
````

## Entities in Home Assistant

All entities created by the bean are created as MQTT entities in Home Assistant and are automatically registered to
the bean's device. This consolidates all entities associated with your bean in a single location within the Home 
Assistant interface, making them easily accessible when viewing the device in the UI.

````java
@SmartBeanDef(beanDevice = @BeanDevice(type = BeanDevice.EntityType.BINARY_SENSOR))
public class FibonacciGenerator implements SmartBean {

  @State
  private int current;
  @State
  private int previous;

  @Provided(entityId = "sensor.fibonacci_sequence", icon = "mdi:rabbit", friendlyName = "Fibonacci Sequence")
  private ProvidedSensor fibonacciSensor;

  @Provided(entityId = "button.reset_fibonacci_sequence", friendlyName = "Reset")
  private ProvidedButton resetSequence;

  @Provided(entityId = "button.calculate_next_fibonacci", friendlyName = "Calculate next", icon = "mdi:calculator")
  private ProvidedButton calculateNext;

  private void updateSensor() {
    fibonacciSensor.setState(current);
  }

  @OnButtonPressed("resetSequence")
  public void resetSequence() {
    current = 1;
    previous = 1;
    updateSensor();
  }

  @OnButtonPressed("calculateNext")
  public void calculateNext() {
    int tmp = current;
    current += previous;
    previous = tmp;
    updateSensor();
  }

  @Override
  public void readState(StateReader reader) {
    if(current == 0) {
      current = 1;
      previous = 1;
    }
    updateSensor();
  }
}
````

The created device of this example SmartBean looks like this in Home Assistant:

![Fibonacci sequence in Home Assistant](/img/screenshots/fibonacci_sequence.png)

Have a look at the [provided entities reference](../reference/provided) for more information about the different types 
of provided entities and how to use them.