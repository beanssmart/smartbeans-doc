---
sidebar_position: 30
---

# Register to triggers

A fundamental concept for automations in Home Assistant is the implementation of [triggers](https://www.home-assistant.io/docs/automation/trigger/). 
Home Assistant supports numerous trigger types that activate when specific events occur. For instance, a state trigger 
is activated when an entity changes its state. SmartBeans allows you to respond to these triggers efficiently. For 
commonly used triggers, you can simply annotate a method in your SmartBean with the appropriate annotation, and
SmartBeans will automatically invoke this method when the corresponding trigger fires. For more complex scenarios, 
SmartBeans also provides a generic approach that enables you to register listeners for any trigger type.

## Using annotations

First, let's examine the annotation-based approach. The first method in the following example is invoked when the state
of the `binary_sensor.motion_kitchen` entity changes to `on`, indicating motion detection in the kitchen. The second 
method is triggered when the entity state changes to `off`, meaning that motion is no longer detected in the kitchen.

````java
public class KitchenMotionControl implements SmartBean {

  @Entity("light.kitchen_ceiling")
  private Light ceilingLight;

  @OnStateTrigger(entity = "binary_sensor.motion_kitchen", to = "on")
  public void onMotionDetected() {
    ceilingLight.turnOn();
  }
  
  @OnStateTrigger(entity = "binary_sensor.motion_kitchen", to = "off")
  public void onNoMotionDetected() {
    ceilingLight.turnOff();
  }
}
````

Each trigger also provides an event, which can be passed to the annotated method. This event contains information about
the occurred event, in this case for example the previous and current state of the entity. With this information, you 
can combine both methods into a single implementation:

````java
public class KitchenMotionControl implements SmartBean {

  @Entity("light.kitchen_ceiling")
  private Light ceilingLight;

  @OnStateTrigger(entity = "binary_sensor.motion_kitchen")
  public void onMotionDetected(StateEvent event) {
    BinarySensor.State newState = BinarySensor.State.fromValue(event.getNewState().getState());
    if(newState == BinarySensor.State.ON) {
      ceilingLight.turnOn();
    }
    else {
      ceilingLight.turnOff();
    }
  }
}
````

Have a look at the [trigger reference documentation](../reference/trigger) for more information about supported trigger
types, their specific annotations, and the corresponding event objects.

## Generic trigger support

SmartBeans currently does not provide native support for all Home Assistant trigger types. In such cases, you can 
leverage the generic API approach to register listeners for any unsupported trigger type. For instance, the 
[calendar trigger](https://www.home-assistant.io/docs/automation/trigger/#calendar-trigger) lacks direct support in the
current implementation, but you can still register a listener for this trigger type by utilizing the `SmartBeans` API
with the `registerTrigger()` method as demonstrated below:

````java
public class ASampleBean implements SmartBean {
  @Override
  public void init(SmartBeans sb) {
    TriggerRegistration<TriggerEvent> calendarTrigger = sb.registerTrigger(
        Triggers.generic("calendar")
            .withData(data -> data
                .setAttribute("event", "start")
                .setAttribute("entity_id", "calendar.light_schedule")
                .setAttribute("offset", "-00:05:00")
            )
    );
    calendarTrigger.onTrigger(this::onCalendarTrigger);
  }
  
  public void onCalendarTrigger(TriggerEvent evt) {
    //this event is also generic and provides all data from the fired trigger
    evt.getTriggerData().asString("event");
  }
}
````
