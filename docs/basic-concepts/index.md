# SmartBean

A SmartBean is the core element in SmartBeans. It represents a single automation context, such as "motion-controlled
lighting". Within a single Java class, it bundles all automations (similar to Home Assistant automations) that handle 
this specific context. Each Home Assistant automation becomes a method in this class. For example, when controlling a 
light with a motion sensor, you typically need multiple automations: one to turn the light on when movement is detected, 
and another to turn it off.

Home Assistant uses triggers that fire when specific events occur, allowing you to build automations that respond with
defined actions. For example, when a motion sensor's state changes, it can trigger an action to turn on a light.

In a SmartBean, you can connect any public method to a trigger using annotations that correspond to common Home
Assistant triggers. Actions (or service calls) are executed through Java objects, such as calling turnOn() on a light
entity. These objects can be automatically injected into your SmartBean using annotations.

````java
@Entity("light.kitchen_ceiling")
private Light light;

@OnStateTrigger(entity = "binary_sensor.motion_kitchen", to = "on")
public void onMotionChange() {
  light.turnOn();
}
````
