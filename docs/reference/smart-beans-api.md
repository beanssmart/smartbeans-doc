---
sidebar_position: 50
---

# `SmartBeans` API

The `SmartBeans` object acts as the central programming interface for a SmartBean. While annotations cover most common 
use cases, the `SmartBeans` API provides direct access for advanced scenarios. It enables you to programmatically 
interact with any Home Assistant service or trigger, including those not explicitly supported by the framework's 
annotations.  

To use the `SmartBeans` API, simply declare a field of type `SmartBeans` in your bean class. The framework will 
automatically inject the appropriate instance into that field.  

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;
}
````

The following is a complete reference of all methods provided by the `SmartBeans` API, organized by task category.

## Framework Utilities

This section provides utility methods that are commonly used within a SmartBean. These include access to the current 
time, retrieving configuration parameters, logging messages, and interacting with other beans.

<hr className="subtle-hr" />

### getNow

`getNow()`

Returns the current time as an `Instant`. Unlike `Instant.now()`, this method allows the time to be controlled or 
manipulated in JUnit test cases, making your tests deterministic and reproducible.

**Returns**  
- `Instant`: The current timestamp.  

**Example**  

````java
Instant now = sb.getNow();
System.out.println("Current time: " + now);
````

<hr className="subtle-hr" />

### getParameter

`getParameter(String name)`

Retrieves a [parameter of the bean](../basic-concepts/instances#bean-parameters) by name.

| Parameter | Type     | Description            |
|-----------|----------|------------------------|
| name      | `String` | Name of the parameter. |

**Returns**  
- `BeanParameter`: Object representing the parameter, which provides convenience methods for different types.  

**Example**  

````java
BeanParameter threshold = sb.getParameter("motionThreshold");
System.out.println("Threshold: " + threshold.asInt(0));
````

<hr className="subtle-hr" />

### log

`log(String message)`
`log(String template, Object... args)`
`log(Supplier<String> message)`

Logs a message to the SmartBeans logging system. Every bean has its own logging space, where all messages are collected.
Supports plain messages, formatted messages, or lazy evaluation via a `Supplier`.

| Parameter | Type               | Description                                                                    |
|-----------|--------------------|--------------------------------------------------------------------------------|
| message   | `String`           | Message to be logged.                                                          |
| template  | `String`           | Message template to be logged, like the `printf()` mechanism in standard Java. |
| args      | `Object...`        | Arguments for the message template.                                            |
| message   | `Supplier<String>` | Message with lazy evaluation, useful to avoid unnecessary computation.         |

**Example**  

````java
sb.log("Motion detected in kitchen");
sb.log("Temperature is %d°C", 23);
sb.log(() -> "Dynamic log message with value " + someValue);
````

<hr className="subtle-hr" />

### getBean

`getBean(Class<Bean> beanType)`
`getBean(Class<Bean> beanType, String beanName)`

Retrieves a [proxy to another SmartBean](../basic-concepts/access-other), allowing method calls in the context of the
target bean's thread. This is useful to safely interact with other beans without directly invoking their methods.

| Parameter | Type          | Description                                                                                   |
|-----------|---------------|-----------------------------------------------------------------------------------------------|
| beanType  | `Class<Bean>` | Class type of the target bean.                                                                |
| beanName  | `String`      | Optional name of the target bean, required only if multiple instances of the same bean exist. |

**Returns**  
- `BeanProxy<Bean>`: Proxy object for invoking methods on the target bean.  

**Example**  

````java
BeanProxy<KitchenMotionControl> kitchen = sb.getBean(KitchenMotionControl.class);
kitchen.invoke(KitchenMotionControl::onMotion);
````

<hr className="subtle-hr" />

### callAsync

`invokeAsync(Callable<?> backgroundTask)`  
`invokeAsync(Runnable backgroundTask)`

Executes the provided task in the background, outside the bean thread. This is intended for time-consuming operations
(for example, web service calls) that should not block the entire bean. All other bean methods, like triggers and
callbacks, are executed in parallel with the background task. 

Be careful when accessing bean state from the background process. To avoid issues, it is recommended **not to read or
write bean state** directly from the background task. If you need to use bean state, create a copy before starting the
task. If you want to update bean state, use the callback mechanism provided by the `onFinished()` method of the returned
`BeanFuture` object, as this callback always runs in the bean thread.

| Parameter      | Type          | Description                                                                                          |
|----------------|---------------|------------------------------------------------------------------------------------------------------|
| backgroundTask | `Callable<?>` | Background task as a Java `Callable`, if you need to return a result.                                |
| backgroundTask | `Runnable`    | Background task as a Java `Runnable`, if no result is needed.                                        |

**Returns**  
- `BeanFuture<?>`: Future object to access the result and register callbacks for when the background task finishes.  

**Example**  

````java
BeanFuture<WeatherData> future = sb.callAsync(() -> {
  WeatherClient client = new WeatherClient(serviceUrl);
  return client.getCurrentWeather();
});
future.onFinished(data -> sb.log("Weather forecast is " + data.getForecast()));
````

<hr className="subtle-hr" />

## Actions and Triggers

This section covers methods to interact with Home Assistant dynamically, such as registering triggers, and calling 
services.

<hr className="subtle-hr" />

### registerTrigger

`registerTrigger(TriggerDefinition<Event> triggerDefinition)`

Registers a trigger in Home Assistant. This method can be used as an alternative to trigger annotations, as described 
in [Register to triggers](../basic-concepts/trigger). The trigger definition can be created using the `Triggers` factory
class, which provides methods for different trigger types as described in the [trigger reference](trigger).

| Parameter         | Type                       | Description                                                               |
|-------------------|----------------------------|---------------------------------------------------------------------------|
| triggerDefinition | `TriggerDefinition<Event>` | Definition of the trigger, typically created with the `Triggers` factory. |

**Returns**  
- `TriggerRegistration<Event>`: Object to manage the registration and unregister the trigger if needed.

**Example**  

````java
TriggerRegistration<StateEvent> registration = sb.registerTrigger(
    Triggers.state()
        .ofEntity("binary_sensor.motion_kitchen")
        .from("off")
        .to("on")
);
// register callback
registration.onTrigger(evt -> handleMotion());
// unregister trigger
registration.unregister();
````

<hr className="subtle-hr" />

### callService

`callService(Service service)`

Calls any Home Assistant service. This method is useful if the service is not available through the entity objects 
provided by SmartBeans. The `Service` parameter object **must** be created using the `Service.name()` factory method, which 
initializes a service call with a given service name. After creation, the returned `Service` object can be further 
configured with targets and additional data, such as attributes or parameters, before calling the service.

| Parameter | Type      | Description                                                                                                          |
|-----------|-----------|----------------------------------------------------------------------------------------------------------------------|
| service   | `Service` | Service to call in Home Assistant. Use the `Service.name()` factory method to create and configure the service call. |

**Example**  

````java
sb.callService(
    Service.name("light.turn_on")                               // Create a service call for "light.turn_on"
        .onTarget(Target.entity("light.kitchen_ceiling"))       // Set the service target (entity, area, or device)
        .withData(atts -> atts.setAttribute("brightness", 120)) // Configure additional attributes
);
````

This example shows that you can first create a Service object with `Service.name()`, then set its target(s) via `onTarget()` 
(entities, areas, or devices), and finally configure additional parameters via `withData()` by passing a builder for all 
attributes. Once fully configured, passing the Service object to `callService()` executes the action in Home Assistant.

<hr className="subtle-hr" />

### createTimer

`createTimer(Builder<TimerDefinition> definition)`

Creates a [timer](../basic-concepts/timer) to execute actions after a specified duration. This method only creates the
timer; it must be started explicitly unless the autostart feature is enabled.

| Parameter  | Type                       | Description                                                                          |
|------------|----------------------------|--------------------------------------------------------------------------------------|
| definition | `Builder<TimerDefinition>` | Builder for defining the timer, including interval, behavior, and actions on events. |

**Returns**  
- `Timer`: Timer object that can be started, cancelled, or restarted.

**Example**  

````java
Timer timer = sb.createTimer(timerDefinition -> timerDefinition
    .setInterval(Duration.ofSeconds(60))   // Set timer interval
    .onElapsed(evt -> turnOffLight())      // Action to perform when timer elapses
);
timer.start(); // Start the timer
````

See [timer documentation](../basic-concepts/timer) for more details and additional features.

<hr className="subtle-hr" />

## Entity Access
These methods allow you to access and work with Home Assistant entities.

<hr className="subtle-hr" />

### getBinarySensor

`getBinarySensor(String entityId)`

Creates a [BinarySensor](entities/binary-sensor) object connected to the corresponding Home Assistant entity.  

| Parameter | Type     | Description                                       |
|-----------|----------|---------------------------------------------------|
| entityId  | `String` | Entity ID of the binary sensor in Home Assistant. |

**Returns**  
- `BinarySensor`: Interface to retrieve the current state, including attributes, of the binary sensor.

**Example**  

````java
BinarySensor motion = sb.getBinarySensor("binary_sensor.motion_kitchen");

// Check for motion
if (motion.isOn()) {
    // Handle motion detected
}
````

<hr className="subtle-hr" />

### getCamera

`getCamera(String entityId)`

Creates a [Camera](entities/camera) object connected to the corresponding Home Assistant entity.  

| Parameter | Type     | Description                                |
|-----------|----------|--------------------------------------------|
| entityId  | `String` | Entity ID of the camera in Home Assistant. |

**Returns**  
- `Camera`: Interface to retrieve the current state, including attributes, of the camera and call services on it.

**Example**  

````java
Camera camera = sb.getCamera("camera.frontdoor");

// Turn camera on
camera.turnOn();
````

<hr className="subtle-hr" />

### getClimate

`getClimate(String entityId)`

Creates a [Climate](entities/climate) object connected to the corresponding Home Assistant entity.  

| Parameter | Type     | Description                                 |
|-----------|----------|---------------------------------------------|
| entityId  | `String` | Entity ID of the climate in Home Assistant. |

**Returns**  
- `Climate`: Interface to retrieve the current state, including attributes, of the climate and call services on it.

**Example**  

````java
Climate airCondition = sb.getClimate("climate.ac_livingroom");

// Set HVAC mode to AUTO
airCondition.setHvacMode(Climate.HvacMode.AUTO);
````

<hr className="subtle-hr" />

### getCover

`getCover(String entityId)`

Creates a [Cover](entities/cover) object connected to the corresponding Home Assistant entity.  

| Parameter | Type     | Description                               |
|-----------|----------|-------------------------------------------|
| entityId  | `String` | Entity ID of the cover in Home Assistant. |

**Returns**  
- `Cover`: Interface to retrieve the current state, including attributes, of the cover and call services on it.

**Example**  

````java
Cover officeCover = sb.getCover("cover.office");

// Set the cover position to 50%
officeCover.setPosition(50);
````

<hr className="subtle-hr" />

### getLight

`getLight(String entityId)`

Creates a [Light](entities/light) object connected to the corresponding Home Assistant entity.  

| Parameter | Type     | Description                               |
|-----------|----------|-------------------------------------------|
| entityId  | `String` | Entity ID of the light in Home Assistant. |

**Returns**  
- `Light`: Interface to retrieve the current state, including attributes, of the light and call services on it.

**Example**  

````java
Light ceiling = sb.getLight("light.kitchen_ceiling");

// Turn on the light with brightness 120 and 0.5s transition
ceiling.turnOn(LightAttr.brightness(120), LightAttr.transition(0.5));
````

<hr className="subtle-hr" />

### getLock

`getLock(String entityId)`

Creates a [Lock](entities/lock) object connected to the corresponding Home Assistant entity.  

| Parameter | Type     | Description                              |
|-----------|----------|------------------------------------------|
| entityId  | `String` | Entity ID of the lock in Home Assistant. |

**Returns**  
- `Lock`: Interface to retrieve the current state, including attributes, of the lock and call services on it.

**Example**  

````java
Lock frontDoorLock = sb.getLock("lock.frontdoor");

// Open the lock
frontDoorLock.open();
````

<hr className="subtle-hr" />

### getMediaPlayer

`getMediaPlayer(String entityId)`

Creates a [MediaPlayer](entities/media-player) object connected to the corresponding Home Assistant entity.  

| Parameter | Type     | Description                                      |
|-----------|----------|--------------------------------------------------|
| entityId  | `String` | Entity ID of the media player in Home Assistant. |

**Returns**  
- `MediaPlayer`: Interface to retrieve the current state, including attributes, of the media player and call services on it.

**Example**  

````java
MediaPlayer speaker = sb.getMediaPLayer("media_player.livingroom_speaker");

// Set volume to 30%
speaker.setVolumePercent(30);
````

<hr className="subtle-hr" />

### getScene

`getScene(String entityId)`

Creates a [Scene](entities/scene) object connected to the corresponding Home Assistant entity.  

| Parameter | Type     | Description                               |
|-----------|----------|-------------------------------------------|
| entityId  | `String` | Entity ID of the scene in Home Assistant. |

**Returns**  
- `Scene`: Interface to activate the scene.

**Example**  

````java
Scene cosy = sb.getScene("scene.cosy_lights");

// Activate the scene with transition time 1 second
cosy.turnOn(1);
````

<hr className="subtle-hr" />

### getSensor

`getSensor(String entityId)`

Creates a [Sensor](entities/sensor) object connected to the corresponding Home Assistant entity.  

| Parameter | Type     | Description                                |
|-----------|----------|--------------------------------------------|
| entityId  | `String` | Entity ID of the sensor in Home Assistant. |

**Returns**  
- `Sensor`: Interface to retrieve the current state, including attributes, of the sensor.

**Example**  

````java
Sensor outsideTemp = sb.getSensor("sensor.outside_temperature");

// Check if the temperature is above 20°C
if(outsideTemp.asInteger() > 20) {
  // perform some action
}
````

<hr className="subtle-hr" />

### getSwitch

`getSwitch(String entityId)`

Creates a [Switch](entities/switch) object connected to the corresponding Home Assistant entity.  

| Parameter | Type     | Description                                |
|-----------|----------|--------------------------------------------|
| entityId  | `String` | Entity ID of the switch in Home Assistant. |

**Returns**  
- `Switch`: Interface to retrieve the current state, including attributes, of the switch and call services on it.

**Example**  

````java
Switch humidifier = sb.getSwitch("switch.humidifier");

// Turn the switch on
humidifier.turnOn();
````

<hr className="subtle-hr" />

### getVacuum

`getVacuum(String entityId)`

Creates a [Vacuum](entities/vacuum) object connected to the corresponding Home Assistant entity.  

| Parameter | Type     | Description                                |
|-----------|----------|--------------------------------------------|
| entityId  | `String` | Entity ID of the vacuum in Home Assistant. |

**Returns**  
- `Vacuum`: Interface to retrieve the current state, including attributes, of the vacuum and call services on it.

**Example**  

````java
Vacuum vacuum = sb.getVacuum("vacuum.robbie");

// Start the vacuum
vacuum.start();
````

<hr className="subtle-hr" />

### getState

`getState(String entityId)`

Gets the state of any entity, regardless of its domain. This can be used to retrieve the state of entities in domains
that are not (yet) implemented in SmartBeans.

| Parameter | Type     | Description                                |
|-----------|----------|--------------------------------------------|
| entityId  | `String` | Entity ID of the entity in Home Assistant. |

**Returns**
- `Entity<?>`: Interface to retrieve the current state, including attributes, of the entity.

**Example**

```java
Entity<?> birthdaysCalendar = sb.getState("calendar.birthdays");
System.out.println("Next birthday is: " + birthdaysCalendar.getStateAsString());
````

<hr className="subtle-hr" />

## Helper Access
These methods allow you to access and work with Home Assistant helpers.

<hr className="subtle-hr" />

### getCounter

`getCounter(String entityId)`

Creates a [Counter](entities/helper#counter) object connected to the corresponding Home Assistant entity.  

| Parameter | Type     | Description                                 |
|-----------|----------|---------------------------------------------|
| entityId  | `String` | Entity ID of the counter in Home Assistant. |

**Returns**  
- `Counter`: Interface to retrieve current value of the counter and call services on it.

**Example**  

````java
// Get the counter for tracking sunny days
Counter sunnyDaysCounter = sb.getCounter("counter.sunny_days");

// Increment the counter
sunnyDaysCounter.increment();
````

<hr className="subtle-hr" />

### getInputBoolean

`getInputBoolean(String entityId)`

Creates a [InputBoolean](entities/helper#input-boolean) object connected to the corresponding Home Assistant entity.  

| Parameter | Type     | Description                                       |
|-----------|----------|---------------------------------------------------|
| entityId  | `String` | Entity ID of the input boolean in Home Assistant. |

**Returns**  
- `InputBoolean`: Interface to retrieve or set the current value of the input boolean.

**Example**  

````java
InputBoolean atHomeBoolean = sb.getInputBoolean("input_boolean.at_home");

// Check if the boolean is on
if (atHomeBoolean.isOn()) {
  // do something
}
````

<hr className="subtle-hr" />

### getInputDateTime

`getInputDateTime(String entityId)`

Creates a [InputDateTime](entities/helper#input-date-time) object connected to the corresponding Home Assistant entity.  

| Parameter | Type     | Description                                        |
|-----------|----------|----------------------------------------------------|
| entityId  | `String` | Entity ID of the input datetime in Home Assistant. |

**Returns**  
- `InputDateTime`: Interface to retrieve or set the current value of the input datetime.

**Example**  

````java
InputDateTime alarmClockDateTime = sb.getInputDateTime("input_datetime.alarm_clock");

// Set the time to 07:15
alarmClockDateTime.setValue(LocalTime.of(7, 15));
````

<hr className="subtle-hr" />

### getInputNumber

`getInputNumber(String entityId)`

Creates a [InputNumber](entities/helper#input-number) object connected to the corresponding Home Assistant entity.  

| Parameter | Type     | Description                                      |
|-----------|----------|--------------------------------------------------|
| entityId  | `String` | Entity ID of the input number in Home Assistant. |

**Returns**  
- `InputNumber`: Interface to retrieve or set the current value of the input number and call services on it.

**Example**  

````java
InputNumber wateringDuration = sb.getInputNumber("input_number.garden_watering_duration");

// Set duration to 15 minutes
wateringDuration.setValue(15);
````

<hr className="subtle-hr" />

### getInputSelect

`getInputSelect(String entityId)`

Creates a [InputSelect](entities/helper#input-select) object connected to the corresponding Home Assistant entity.  

| Parameter | Type     | Description                                      |
|-----------|----------|--------------------------------------------------|
| entityId  | `String` | Entity ID of the input select in Home Assistant. |

**Returns**  
- `InputSelect`: Interface to retrieve or set the current value of the input select and call services on it.

**Example**  

````java
InputSelect color = sb.getInputSelect("input_select.favorite_color");

// Select the first option
color.selectFirst();
````

<hr className="subtle-hr" />

### getInputText

`getInputText(String entityId)`

Creates a [InputText](entities/helper#input-text) object connected to the corresponding Home Assistant entity.  

| Parameter | Type     | Description                                    |
|-----------|----------|------------------------------------------------|
| entityId  | `String` | Entity ID of the input text in Home Assistant. |

**Returns**  
- `InputText`: Interface to retrieve or set the current value of the input text.

**Example**  

````java
InputText greetingPhrase = sb.getInputText("input_text.greeting_phrase");

// Set the greeting phrase
greetingPhrase.setValue("Hello!");
````

<hr className="subtle-hr" />

## Provide Entities

These methods create [provided entities](../basic-concepts/provided) for your SmartBean.

<hr className="subtle-hr" />

### provideBinarySensor

`provideBinarySensor(String uniqueId, String entityId)`  
`provideBinarySensor(String uniqueId, String entityId, Builder<BinarySensorBuilder> builder)`  
`provideBinarySensor(BinarySensorDefinition definition)`

Creates a [provided binary sensor](provided/binary-sensor) for the bean device and returns an object to control
the created binary sensor in Home Assistant.

| Parameter   | Type                           | Description                                                                                                                        |
|-------------|--------------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| uniqueId    | `String`                       | Unique ID for the entity within the scope of the bean.                                                                             |
| entityId    | `String`                       | Entity ID of the created entity in Home Assistant.                                                                                 |
| builder     | `Builder<BinarySensorBuilder>` | Builder for configuring the properties of the created entity in Home Assistant.                                                    |
| definition  | `BinarySensorDefinition`       | Any custom object implementing the `BinarySensorDefinition` interface, which defines all properties of the provided binary sensor. |

**Returns**  
- `ProvidedBinarySensor`: Object to control the created provided binary sensor, including setting its state in Home Assistant.

**Example**  

````java
ProvidedBinarySensor diskSpaceWarning = sb.provideBinarySensor(
    "diskSpaceWarning",
    "binary_sensor.warning_disk_space",
    def -> def
        .setFriendlyName("Diskspace low")
        .setDeviceClass("warning")
);

diskSpaceWarning.setState(isDiskSpaceLow() ? State.ON : State.OFF);
````

<hr className="subtle-hr" />

### provideSensor

`provideSensor(String uniqueId, String entityId)`  
`provideSensor(String uniqueId, String entityId, Builder<SensorBuilder> builder)`  
`provideSensor(SensorDefinition definition)`

Creates a [provided sensor](provided/sensor) for the bean device and returns an object to control the created sensor
in Home Assistant.

| Parameter   | Type                     | Description                                                                                                           |
|-------------|--------------------------|-----------------------------------------------------------------------------------------------------------------------|
| uniqueId    | `String`                 | Unique ID for the entity within the scope of the bean.                                                                |
| entityId    | `String`                 | Entity ID of the created entity in Home Assistant.                                                                    |
| builder     | `Builder<SensorBuilder>` | Builder for configuring the properties of the created entity in Home Assistant.                                       |
| definition  | `SensorDefinition`       | Any custom object implementing the `SensorDefinition` interface, which defines all properties of the provided sensor. |

**Returns**  
- `ProvidedSensor`: Object to control the created provided sensor, including setting its state in Home Assistant.

**Example**  

````java
ProvidedSensor freeDiskSpace = sb.provideSensor(
    "diskSpace",
    "sensor.free_disk_space",
    def -> def
        .setFriendlyName("Free diskspace")
        .setDeviceClass("data_size")
        .setUnitOfMeasurement("GB")
);

// Update the sensor state
freeDiskSpace.setState(getFreeDiskSpace());
````

<hr className="subtle-hr" />

### provideButton

`provideButton(String uniqueId, String entityId)`  
`provideButton(String uniqueId, String entityId, Builder<ButtonBuilder> builder)`  
`provideButton(ButtonDefinition definition)`

Creates a [provided button](provided/button) for the bean device and returns an object to register listeners
to the created button in Home Assistant.

| Parameter   | Type                     | Description                                                                                                           |
|-------------|--------------------------|-----------------------------------------------------------------------------------------------------------------------|
| uniqueId    | `String`                 | Unique ID for the entity within the scope of the bean.                                                                |
| entityId    | `String`                 | Entity ID of the created entity in Home Assistant.                                                                    |
| builder     | `Builder<ButtonBuilder>` | Builder for configuring the properties of the created entity in Home Assistant.                                       |
| definition  | `ButtonDefinition`       | Any custom object implementing the `ButtonDefinition` interface, which defines all properties of the provided button. |

**Returns**  
- `ProvidedButton`: Object that can be used to register listeners for button presses in Home Assistant.

**Example**  

```java
ProvidedButton calculateButton = sb.provideButton(
    "calc",
    "button.calculate_disk_space",
    def -> def.setFriendlyName("Calculate Disk Space")
);

// Register a listener for button presses
calculateButton.onPressed(evt -> calculateDiskSpace());
````

<hr className="subtle-hr" />

### provideSwitch

`provideSwitch(String uniqueId, String entityId)`  
`provideSwitch(String uniqueId, String entityId, Builder<SwitchBuilder> builder)`  
`provideSwitch(SwitchDefinition definition)`

Creates a [provided switch](provided/switch) for the bean device and returns an object to set the state and register
listeners to the created switch in Home Assistant.

| Parameter   | Type                     | Description                                                                                                           |
|-------------|--------------------------|-----------------------------------------------------------------------------------------------------------------------|
| uniqueId    | `String`                 | Unique ID for the entity within the scope of the bean.                                                                |
| entityId    | `String`                 | Entity ID of the created entity in Home Assistant.                                                                    |
| builder     | `Builder<SwitchBuilder>` | Builder for configuring the properties of the created entity in Home Assistant.                                       |
| definition  | `SwitchDefinition`       | Any custom object implementing the `SwitchDefinition` interface, which defines all properties of the provided switch. |

**Returns**  
- `ProvidedSwitch`: Object that can be used to set the state of the created switch and register listeners for state changes in Home Assistant.

**Example**  

```java
ProvidedSwitch autoMode = sb.provideSwitch(
    "auto-mode",
    "switch.auto_mode",
    def -> def.setFriendlyName("Automatic Mode")
);

//set state
autoMode.setState(State.ON);

// Register a listener when switch is turened on
autoMode.onTurnOn(evt -> enableAutoMode());
````

<hr className="subtle-hr" />

### provideLight

`provideLight(String uniqueId, String entityId)`  
`provideLight(String uniqueId, String entityId, Builder<LightBuilder> builder)`  
`provideLight(LightDefinition definition)`

Creates a [provided light](provided/light) for the bean device and returns an object to set the state and register
listeners to the created light in Home Assistant.

| Parameter   | Type                    | Description                                                                                                         |
|-------------|-------------------------|---------------------------------------------------------------------------------------------------------------------|
| uniqueId    | `String`                | Unique ID for the entity within the scope of the bean.                                                              |
| entityId    | `String`                | Entity ID of the created entity in Home Assistant.                                                                  |
| builder     | `Builder<LightBuilder>` | Builder for configuring the properties of the created entity in Home Assistant.                                     |
| definition  | `SwitchDefinition`      | Any custom object implementing the `LightDefinition` interface, which defines all properties of the provided light. |

**Returns**  
- `ProvidedLight`: Object that can be used to set the state of the created light and register listeners for state changes in Home Assistant.

**Example**  

```java
ProvidedLight customLight = sb.provideLight(
    "custom",
    "light.custom",
    def -> def.setFriendlyName("Custom light")
);

//set state
customLight.setState(State.ON);

// Register a listener when switch is turened on
customLight.onTurnOn(evt -> turnOnLight());
````

<hr className="subtle-hr" />

## Config Entities

These methods create [config entities](../basic-concepts/config) for your SmartBean.

<hr className="subtle-hr" />

### getConfigBoolean

`getConfigBoolean(String name, boolean defaultValue)`
`getConfigBoolean(String name, boolean defaultValue, Builder<ConfigBooleanBuilder> builder)`

Creates a [configuration boolean](config/boolean) for this bean, which can be used to control a true/false value through
a Home Assistant switch entity.

| Parameter    | Type                            | Description                                                                     |
|--------------|---------------------------------|---------------------------------------------------------------------------------|
| name         | `String`                        | Name of the configuration value, must be unique within the scope of the bean.   |
| defaultValue | `boolean`                       | The default value of the configuration.                                         |
| builder      | `Builder<ConfigBooleanBuilder>` | Builder for configuring the properties of the created entity in Home Assistant. |

**Returns**  
- `ConfigBoolean`: Object to query the current configured boolean value.

**Example**  

````java
// Create a configuration boolean to enable or disable motion detection
ConfigBoolean motionDetection = sb.getConfigBoolean("motionDetection", true, def -> def
    .setFriendlyName("Motion Detection")
    .setIcon("mdi:motion-sensor")
);

// Check the current value
if (motionDetection.isOn()) {
    // Motion detection is enabled
}
````

<hr className="subtle-hr" />

### getConfigNumber

`getConfigNumber(String name, int defaultValue)`  
`getConfigNumber(String name, int defaultValue, Builder<ConfigNumberBuilder> builder)`  
`getConfigNumber(String name, double defaultValue)`
`getConfigNumber(String name, double defaultValue, Builder<ConfigNumberBuilder> builder)`

Creates a [configuration number](config/number) for this bean, which can be used to control a numeric value through a
Home Assistant number entity.

| Parameter    | Type                           | Description                                                                     |
|--------------|--------------------------------|---------------------------------------------------------------------------------|
| name         | `String`                       | Name of the configuration value, must be unique within the scope of the bean.   |
| defaultValue | `int` / `double`               | The default value of the configuration.                                         |
| builder      | `Builder<ConfigNumberBuilder>` | Builder for configuring the properties of the created entity in Home Assistant. |

**Returns**  
- `ConfigNumber`: Object to query the current configured value.

**Example**  

````java
// Create a configuration number for the target temperature
ConfigNumber targetTemperature = sb.getConfigNumber("targetTemp", 20, def -> def
    .setUnitOfMeasurement("°C")
    .setMin(10)
    .setMax(30)
    .setStep(0.5)
    .setDisplayMode(DisplayMode.SLIDER)
);
````
<hr className="subtle-hr" />

### getConfigDuration

`getConfigDuration(String name, Duration defaultValue)`
`getConfigDuration(String name, Duration defaultValue, Builder<ConfigDurationBuilder> builder)`

Creates a [configuration duration](config/duration) for this bean, which can be used to control a duration value through
a Home Assistant number entity. The duration is expressed in the unit specified by `unitOfMeasurement` (default is seconds).

| Parameter    | Type                             | Description                                                                     |
|--------------|----------------------------------|---------------------------------------------------------------------------------|
| name         | `String`                         | Name of the configuration value, must be unique within the scope of the bean.   |
| defaultValue | `Duration`                       | The default duration value.                                                     |
| builder      | `Builder<ConfigDurationBuilder>` | Builder for configuring the properties of the created entity in Home Assistant. |

**Returns**  
- `ConfigDuration`: Object to query the current configured duration, with convenience methods for different units and types.

**Example**  

````java
// Create a configuration duration for garden watering
ConfigDuration wateringDuration = sb.getConfigDuration("wateringDuration", Duration.ofMinutes(15), def -> def
    .setUnitOfMeasurement(DurationUnit.MINUTES)  // minutes
    .setMin(Duration.ofMinutes(5))
    .setMax(Duration.ofMinutes(60))
    .setStep(Duration.ofMinutes(5))
    .setFriendlyName("Garden Watering Duration")
);

// Retrieve the value in minutes
long minutes = wateringDuration.toMinutes();
````

<hr className="subtle-hr" />

### getConfigText

`getConfigText(String name, String defaultValue)`
`getConfigText(String name, String defaultValue, Builder<ConfigTextBuilder> builder)`

Creates a [configuration text](config/text) value for this bean, which can be used to control a string value through a 
Home Assistant text entity.

| Parameter    | Type                         | Description                                                                     |
|--------------|------------------------------|---------------------------------------------------------------------------------|
| name         | `String`                     | Name of the configuration value, must be unique within the scope of the bean.   |
| defaultValue | `String`                     | The default string value of the configuration.                                  |
| builder      | `Builder<ConfigTextBuilder>` | Builder for configuring the properties of the created entity in Home Assistant. |

**Returns**  
- `ConfigText`: Object to query the current configured text value.

**Example**  

````java
// Create a configuration text for a custom greeting message
ConfigText greetingMessage = sb.getConfigText("greetingMessage", "Hello!", def -> def
    .setFriendlyName("Greeting Message")
    .setIcon("mdi:message-text")
);

// Retrieve the current value
String message = greetingMessage.getValue();
````

<hr className="subtle-hr" />

### getConfigSelect

`getConfigSelect(String name, String defaultValue, String... values)`  
`getConfigSelect(String name, E defaultValue, Class<E> enumType)`  
`getConfigSelect(String name, T defaultValue, Builder<ConfigSelectBuilder<T>> builder)`

Creates a [configuration select](config/select) value for this bean. A select value allows choosing from a fixed set of
options and is exposed in Home Assistant as a select entity.  

| Parameter    | Type                              | Description                                                                   |
|--------------|-----------------------------------|-------------------------------------------------------------------------------|
| name         | `String`                          | Name of the configuration value, must be unique within the scope of the bean. |
| defaultValue | `String` / `Enum` / generic `T`   | The default selected value.                                                   |
| values       | `String...`                       | Available options (for the string-based method).                              |
| enumType     | `Class<E>`                        | Enum type defining available options (for the enum-based method).             |
| builder      | `Builder<ConfigSelectBuilder<T>>` | Builder for advanced configuration (for the generic method).                  |

**Returns**  
- `ConfigSelect<T>`: Object to query the current configured option.

**Examples**  

````java
ConfigSelect<String> color = sb.getConfigSelect(
    "color", 
    "red", 
    "red", "green", "blue"
);
````

````java
enum Mode { AUTO, COOL, HEAT }

ConfigSelect<Mode> mode = sb.getConfigSelect(
    "hvacMode",
    Mode.AUTO,
    Mode.class
);
````

<hr className="subtle-hr" />

### getConfigRgbColor

`getConfigRgbColor(String name, RgbColor defaultValue)`
`getConfigRgbColor(String name, RgbColor defaultValue, Builder<ConfigRgbColorBuilder> builder)`

Creates a configuration value for a [RGB-based light preset](config/rgb-color). Exposed in Home Assistant as a light 
entity.  

| Parameter    | Type                             | Description                                                                   |
|--------------|----------------------------------|-------------------------------------------------------------------------------|
| name         | `String`                         | Name of the configuration value, must be unique within the scope of the bean. |
| defaultValue | `RgbColor`                       | The default RGB color value.                                                  |
| builder      | `Builder<ConfigRgbColorBuilder>` | Builder for advanced configuration of the color entity.                       |

**Returns**  
- `ConfigRgbColor`: Access to the currently configured RGB color (including brightness) of the light preset, with 
  methods to apply the preset directly to real lights.

**Examples**  

````java
ConfigRgbColor ambientColor = sb.getConfigRgbColor(
    "ambientColor",
    new RgbColor(255, 0, 0), // red
    def -> def.setFriendlyName("Ambient Light Preset")
);
// Query raw values
RgbColor color = ambientColor.getRgbColor();
int brightness = ambientColor.getBrightness();

// Or apply the preset directly to a light
ambientColor.applyTo(sb.getLight("light.kitchen_ceiling"));
````

<hr className="subtle-hr" />

### getConfigRgbwColor

`getConfigRgbwColor(String name, RgbwColor defaultValue)`
`getConfigRgbwColor(String name, RgbwColor defaultValue, Builder<ConfigRgbwColorBuilder> builder)`

Creates a configuration value for a [RGBW-based light preset](config/rgbw-color). Exposed in Home Assistant as a light 
entity.  

| Parameter    | Type                               | Description                                                                   |
|--------------|------------------------------------|-------------------------------------------------------------------------------|
| name         | `String`                           | Name of the configuration value, must be unique within the scope of the bean. |
| defaultValue | `RgbwColor`                        | The default RGBW color value.                                                 |
| builder      | `Builder<ConfigRgbwColorBuilder>`  | Builder for advanced configuration of the color entity.                       |

**Returns**  
- `ConfigRgbwColor`: Access to the currently configured RGBW color (including brightness) of the light preset, with 
  methods to apply the preset directly to real lights.

**Examples**  

````java
ConfigRgbwColor ambientColor = sb.getConfigRgbwColor(
    "ambientColor",
    new RgbwColor(255, 0, 0, 128), // red with half white
    def -> def.setFriendlyName("Ambient RGBW Preset")
);

// Query raw values
RgbwColor color = ambientColor.getRgbwColor();
int brightness = ambientColor.getBrightness();

// Or apply the preset directly to a light
ambientColor.applyTo(sb.getLight("light.living_room_ceiling"));
````

<hr className="subtle-hr" />

### getConfigRgbwwColor

`getConfigRgbwwColor(String name, RgbwwColor defaultValue)`
`getConfigRgbwwColor(String name, RgbwwColor defaultValue, Builder<ConfigRgbwwColorBuilder> builder)`

Creates a configuration value for a [RGBWW-based light preset](config/rgbww-color). Exposed in Home Assistant as a light 
entity.  

| Parameter    | Type                                 | Description                                                                   |
|--------------|--------------------------------------|-------------------------------------------------------------------------------|
| name         | `String`                             | Name of the configuration value, must be unique within the scope of the bean. |
| defaultValue | `RgbwwColor`                         | The default RGBWW color value.                                                |
| builder      | `Builder<ConfigRgbwwColorBuilder>`   | Builder for advanced configuration of the color entity.                       |

**Returns**  
- `ConfigRgbwwColor`: Access to the currently configured RGBWW color (including brightness, warm white and cold white) 
  of the light preset, with methods to apply the preset directly to real lights.

**Examples**  

````java
ConfigRgbwwColor ambientColor = sb.getConfigRgbwwColor(
    "ambientColor",
    new RgbwwColor(255, 200, 150, 128, 64), // warm tone with some WW/CW balance
    def -> def.setFriendlyName("Ambient RGBWW Preset")
);

// Query raw values
RgbwwColor color = ambientColor.getRgbwwColor();
int brightness = ambientColor.getBrightness();

// Or apply the preset directly to a light
ambientColor.applyTo(sb.getLight("light.bedroom_ceiling"));
````

<hr className="subtle-hr" />

### getConfigHsColor

`getConfigHsColor(String name, HsColor defaultValue)`  
`getConfigHsColor(String name, HsColor defaultValue, Builder<ConfigHsColorBuilder> builder)`

Creates a configuration value for a [HS-based light preset](config/hs-color). Exposed in Home Assistant as a light 
entity.  

| Parameter    | Type                               | Description                                                                   |
|--------------|------------------------------------|-------------------------------------------------------------------------------|
| name         | `String`                           | Name of the configuration value, must be unique within the scope of the bean. |
| defaultValue | `HsColor`                          | The default HS (hue/saturation) color value.                                  |
| builder      | `Builder<ConfigHsColorBuilder>`    | Builder for advanced configuration of the color entity.                       |

**Returns**  
- `ConfigHsColor`: Access to the currently configured HS color (including brightness) of the light preset, with methods 
  to apply the preset directly to real lights.

**Examples**  

````java
ConfigHsColor moodColor = sb.getConfigHsColor(
    "moodColor",
    new HsColor(200, 80), // blue tone with strong saturation
    def -> def.setFriendlyName("Mood Light Preset")
);

// Query raw values
HsColor color = moodColor.getHsColor();
int brightness = moodColor.getBrightness();

// Or apply the preset directly to a light
moodColor.applyTo(sb.getLight("light.livingroom_lamp"));
````

<hr className="subtle-hr" />

### getConfigXyColor

`getConfigXyColor(String name, XyColor defaultValue)`  
`getConfigXyColor(String name, XyColor defaultValue, Builder<ConfigXyColorBuilder> builder)`

Creates a configuration value for a [XY-based light preset](config/xy-color). Exposed in Home Assistant as a light 
entity.  

| Parameter    | Type                               | Description                                                                   |
|--------------|------------------------------------|-------------------------------------------------------------------------------|
| name         | `String`                           | Name of the configuration value, must be unique within the scope of the bean. |
| defaultValue | `XyColor`                          | The default XY color value (CIE 1931 color space).                            |
| builder      | `Builder<ConfigXyColorBuilder>`    | Builder for advanced configuration of the color entity.                       |

**Returns**  
- `ConfigXyColor`: Access to the currently configured XY color (including brightness) of the light preset, with methods 
  to apply the preset directly to real lights.

**Examples**  

````java
ConfigXyColor readingColor = sb.getConfigXyColor(
    "readingColor",
    new XyColor(0.323, 0.329), // neutral white
    def -> def.setFriendlyName("Reading Light Preset")
);

// Query raw values
XyColor color = readingColor.getXyColor();
int brightness = readingColor.getBrightness();

// Or apply the preset directly to a light
readingColor.applyTo(sb.getLight("light.office_desk"));
````

<hr className="subtle-hr" />

### getConfigColorTemp

`getConfigColorTemp(String name, int defaultValue)`  
`getConfigColorTemp(String name, int defaultValue, Builder<ConfigColorTempBuilder> builder)`

Creates a configuration value for a [color temperature-based light preset](config/color-temp). Exposed in Home Assistant as a light 
entity.  

| Parameter    | Type                              | Description                                                                   |
|--------------|-----------------------------------|-------------------------------------------------------------------------------|
| name         | `String`                          | Name of the configuration value, must be unique within the scope of the bean. |
| defaultValue | `int`                             | The default color temperature in Kelvin.                                      |
| builder      | `Builder<ConfigColorTempBuilder>` | Builder for advanced configuration of the color temperature entity.           |

**Returns**  
- `ConfigColorTemp`: Access to the currently configured color temperature (including brightness) of the light preset, with methods 
  to apply the preset directly to real lights.

**Examples**  

````java
ConfigColorTemp warmWhite = sb.getConfigColorTemp(
    "warmWhite",
    2700, // 2700K warm white
    def -> def.setFriendlyName("Warm White Light Preset")
);

// Query raw values
int colorTemp = warmWhite.getColorTemp();
int brightness = warmWhite.getBrightness();

// Or apply the preset directly to a light
warmWhite.applyTo(sb.getLight("light.livingroom_ceiling"));
````
