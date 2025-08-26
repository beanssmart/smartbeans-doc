---
sidebar_label: Time pattern
description: Triggers on specific time patterns, like every 5 minutes.
---

# Time pattern trigger

The time pattern trigger enables you to execute actions if the hour, minute or second of the current time matches a 
specific value. It implements the same functionality as the Home Assistant [time pattern trigger](https://www.home-assistant.io/docs/automation/trigger/#time-pattern-trigger),
providing a seamless integration between SmartBeans and Home Assistant.

## Annotating methods

The `@OnTimeTrigger` annotation allows you to invoke any SmartBean method every full hour.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  @OnTimeTrigger(minutes = "0")
  public void onFullHour() {
    sb.log("Kuck-kuck");
  }
}
````

### Intervals

You can also match intervals using a `/` and a number. For example, to match every 5 minutes, you can use the 
following annotation:

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  @OnTimeTrigger(minutes = "/5")
  public void onEvery5Minutes() {
    sb.log("Again 5 minutes are over...");
  }
}
````

### Time events

Your annotated method can optionally include a `TimeEvent` parameter. When the trigger is fired, SmartBeans provides an
event object that gives you access to the event parameters.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  @OnTimeTrigger(minutes = "0")
  public void atFullHour(TimeEvent event) {
    sb.log("Kuck-kuck, its now " + event.getTime());
  }
}
````

## Register triggers programmatically

As an alternative to the annotation-based approach, you can register triggers programmatically using the 
`registerTrigger()` method of the `SmartBeans` API. To register a time pattern trigger, you can use the `timePattern()` 
factory method of the `Triggers` class. This factory provides a fluent API to create the same triggers as with 
annotations. There are two main reasons to choose the programmatic approach over annotations:
1. You need to create triggers with dynamic parameters.
2. You need to register or unregister the trigger at a specific point in time, rather than having it permanently active.

Here is an example of registering and unregistering a trigger with the programmatic approach:

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;
  private TriggerRegistration<TimeEvent> timePatternTriggerRegistration;

  public void registerTimePatternTrigger() {
    timePatternTriggerRegistration = sb.registerTrigger(
      Triggers.timePattern()
          .everyHour(2)
    );
    timePatternTriggerRegistration.onTrigger(this::every2Hours);
  }

  public void unregisterTimePatternTrigger() {
    timePatternTriggerRegistration.unregister();
  }

  private void every2Hours(TimeEvent event) {
    sb.log("Two hours have passed!");
  }
}
````
