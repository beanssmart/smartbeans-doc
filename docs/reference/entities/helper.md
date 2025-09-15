---
sidebar_position: 99
title: Helper
description: Helper entities in Home Assistant.
---

# Home Assistant Helper Entities

Home Assistant provides helper entities that serve two primary purposes:

1. **State Processing**: Helper entities can process and transform states and attributes from other entities. They 
   create new sensors for purposes like aggregation or templating. These helper entities integrate seamlessly with 
   SmartBeans through entity interfaces like [Sensor](./sensor).

2. **Data Management**: Helper entities enable storage and management of data that can be used across the UI and automations.
   For instance, an `input_text` entity allows you to store and manage textual data. SmartBeans provides intuitive interfaces 
   for these helper entities, making them as straightforward to use as any other entity type.

Let's explore the data storing entities in detail and how you can use them in SamrtBeans.

## Counter

The [counter](https://www.home-assistant.io/integrations/counter) helper in Home Assistant is used to count a value in
defined steps. In SmartBeans there is the `Counter` interface to interact with these entities.

````java
public class ASampleBean implements SmartBean {

  @Entity("counter.rainy_days")
  private Counter rainyDays;

  public void todayIsRainy() {
    rainyDays.increment();
  }
}
````

The `Counter` interface has the following methods:

| Method          | Description                                                     |
|-----------------|-----------------------------------------------------------------|
| `getValue()`    | Returns the current value of the counter.                       |
| `getMin()`      | Returns the minimum possible value of the counter.              |
| `getMax()`      | Returns the maximum possible value of the counter.              |
| `getStep()`     | Returns the step this counter increases or decreases his value. |
| `getInitial()`  | Returns the initial value of the counter.                       |
| `increment()`   | Increments the counter by the value of `step`.                  |
| `decrement()`   | Decrements the counter by the value of `step`.                  |
| `setValue(int)` | Sets the value of the counter to the given value.               |
| `reset()`       | Resets the counter to it's initial value.                       |

## Input Boolean

The [input boolean](https://www.home-assistant.io/integrations/input_boolean) helper in Home Assistant is used to create
a boolean value that can be controled by the ui and can be used in automations. In SmartBeans there is the `InputBoolean`
interface to interact with these entities.

````java
public class ASampleBean implements SmartBean {

  @Entity("input_boolean.activate_motion_control")
  private InputBoolean motionControl;

  public void onMotionDetected() {
    if(motionControl.isOn()) {
      //Turn on light.
    }
  }
}
````

The `InputBoolean` interface has the following methods:

| Method          | Description                                       |
|-----------------|---------------------------------------------------|
| `getState()`    | Returns the current state of the helper.          |
| `isOn()`        | Returns `true`, if the input boolean is on.       |
| `turnOn()`      | Sets the value to `true`.                         |
| `turnOff()`     | Sets the value to `false`.                        |
| `toggle()`      | Toggles the value.                                |

## Input Date Time

The [input datetime](https://www.home-assistant.io/integrations/input_datetime) helper in Home Assistant is used to create
a date and/or time value that can be controled by the ui and can be used in automations. In SmartBeans there is the 
`InputDateTime` interface to interact with these entities.

````java
public class ASampleBean implements SmartBean {

  @Entity("input_datetime.alarm_clock_time")
  private InputDateTime alarmClockTime;

  public void setAlarmClock() {
    alarmClockTime.setValue(LocalTime.of(7, 30));
  }
}
````

The `InputDateTime` interface has the following methods:

| Method                 | Description                                                                 |
|------------------------|-----------------------------------------------------------------------------|
| `hasDate()`            | Returns `true`, if this input datetime has a date part.                     |
| `hasTime()`            | Returns `true`, if this input datetime has a time part.                     |
| `getValueAsDate()`     | Returns the current value as a `LocalDate`.                                 |
| `getValueAsTime()`     | Returns the current value as a `LocalTime`.                                 |
| `getValueAsDateTime()` | Returns the current value as a `LocalDateTime`.                             |
| `getValueAsDuration()` | Returns the time part of the current value as a `Duration`.                 |
| `getSecond()`          | Returns the second of the time part.                                        |
| `getMinute()`          | Returns the minute of the time part.                                        |
| `getHour()`            | Returns the hour of the time part.                                          |
| `getDay()`             | Returns the day of the date part.                                           |
| `getMonth()`           | Returns the month of the date part.                                         |
| `getYear()`            | Returns the year of the date part.                                          |
| `setValue()`           | Sets the value to to the given `LocalTime`, `LocalDate` or `LocalDateTime`. |

## Input Number

The [input number](https://www.home-assistant.io/integrations/input_number) helper in Home Assistant is used to create
a number value that can be controled by the ui and can be used in automations. In SmartBeans there is the 
`InputNumber` interface to interact with these entities. In Home Assistant a input number is a floating point value.
The `InputNumber` interface has methods to access all values as `double`, but also has convenient methods for `int` 
values.

````java
public class ASampleBean implements SmartBean {

  @Entity("input_number.bedroom_brightness")
  private InputNumber brightness;

  public void someBeanMethod() {
    setBedroomBrightness(brightness.getValueAsInt());
  }
}
````

The `InputNumber` interface has the following methods:

| Method             | Description                                                                            |
|--------------------|----------------------------------------------------------------------------------------|
| `getValue()`       | Returns current value as `double`.                                                     |
| `getValueAsInt()`  | Returns current value as `int`.                                                        |
| `getMin()`         | Returns the minimum possible value as `double`.                                        |
| `getMinAsInt()`    | Returns the minimum possible value as `int`.                                           |
| `getMax()`         | Returns the maximum possible value as `double`.                                        |
| `getMaxAsInt()`    | Returns the maximum possible value as `int`.                                           |
| `getStep()`        | Returns the step, which is used to change the value on the user interface as `double`. |
| `getStepAsInt()`   | Returns the step, which is used to change the value on the user interface as `int`.    |
| `setValue(double)` | Sets the value to to the given `double`.                                               |
| `setValue(int)`    | Sets the value to to the given `int`.                                                  |
| `increment()`      | Increments the value by the value of `step`.                                           |
| `decrement()`      | Decrements the value by the value of `step`.                                           |

## Input Select

The [input select](https://www.home-assistant.io/integrations/input_select) helper in Home Assistant is used to create
a list of values that can be selected via the ui and can be used in automations. In SmartBeans there is the 
`InputSelect` interface to interact with these entities. The possible values of an input select are defined in Home
Assistant when the entity is created, but can be changed by a service call.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;
  
  @Entity("input_select.color")
  private InputSelect color;

  public void readColor() {
     sb.log("You selected color " + color.getValue() + " out of possible values: " + color.getOptions());
  }
}
````

The `InputSelect` interface has the following methods:

| Method                    | Description                                                                                     |
|---------------------------|-------------------------------------------------------------------------------------------------|
| `getValue()`              | Returns currently selected value.                                                               |
| `getOptions()`            | Returns all possible values.                                                                    |
| `setValue(String)`        | Sets the current value.                                                                         |
| `setOptions(String...)`   | Sets as list with possible values.                                                              |
| `selectOption(String)`    | Convenient for setValue().                                                                      |
| `selectFirst()`           | Selects the first value of all possible values.                                                 |
| `selectLast()`            | Selects the last value of all possible values.                                                  |
| `selectNext(boolean)`     | Selects the next value of all possible values. If parameter is `true`, the value is cycled.     |
| `selectPrevious(boolean)` | Selects the previous value of all possible values. If parameter is `true`, the value is cycled. |

## Input Text

The [input text](https://www.home-assistant.io/integrations/input_text) helper in Home Assistant is used to create
a text value that can be controled by the ui and can be used in automations. In SmartBeans there is the 
`InputText` interface to interact with these entities.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  @Entity("input_text.message")
  private InputText message;

  public void sendMessage() {
     sb.log("sending message: " + message.getValue());
  }
}
````

The `InputText` interface has the following methods:

| Method             | Description                                                   |
|--------------------|---------------------------------------------------------------|
| `getValue()`       | Returns current value.                                        |
| `setValue(String)` | Sets the current value.                                       |
| `getMin()`         | Returns the minimum length of the text to be entered.         |
| `getMax()`         | Returns the maximum length of the text to be entered.         |
| `getPattern()`     | Returns the regular expression all entered values must match. |

## Access Entities Programmatically

Alongside annotations, all helper entities are available programmatically via the `SmartBeans` API's getter methods. 
For example, this can be useful when the entity ID is computed at runtime by business logic and is unavailable at 
compile time.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  public void sendMessage() {
     sb.log("sending message: " + sb.getInputText("input_text.message").getValue());
  }
}
````

import UncachedNote from './_note_uncached_entities.md';

<UncachedNote />