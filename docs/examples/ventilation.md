---
sidebar_position: 20
description: "Ventilation based on humidity, with the option to switch it on manually."
title: Bathroom Ventilation
---

# Bathroom Ventilation

This bean controls the bathroom ventilation in my home.  
It supports two modes:  

- **Automatic mode**: The ventilation starts when the bathroom humidity exceeds a configurable threshold and stops once 
  it falls below that value.  
- **Manual mode**: Activated by a switch, it runs the ventilation for a configurable duration.  

Both the threshold for automatic mode and the duration for manual mode can be adjusted via configuration entities. In 
addition, automatic mode can be enabled or disabled entirely.  

The two modes operate independently of each other. For example, if manual mode is running, the ventilation will not stop
even if the humidity drops below the threshold. Conversely, once the manual duration ends, the ventilation will only
stop if the humidity is also below the set threshold.

This is a screenshot of all entities of the [bean device](../basic-concepts/devices) in Home Assistant.

![Bean device in Home Assistant](/img/examples/ventilation_device.png)

## Source

Here is the complete source file of the bean:

````java showLineNumbers
@SmartBeanDef(
    beanDevice = @BeanDevice(type = BeanDevice.EntityType.SWITCH),
    name = "Badezimmer",
    params = {
        @Param(name = "control_switch", value = "switch.bad_luftung_intern"),
        @Param(name = "manual_switch", value = "switch.bad_luftung"),
        @Param(name = "humidity_sensor", value = "sensor.humidity_bad")
    }
)
public class LueftungSteuerung implements SmartBean {

  private SmartBeans sb;

  @State
  private boolean automatic = false;
  @State
  private boolean manual = false;

  @State(nested = NestedMode.FLAT)
  @TimerDef
  private Timer manualTimer;

  @Entity("${control_switch}")
  private Switch controlSwitch;

  @State
  @Config(friendlyName = "Auto-Mode", icon = "mdi:fan-auto", value = "true")
  private ConfigBoolean automaticMode;

  @State
  @Config(
      friendlyName = "Grenzwert", icon = "mdi:water-percent",
      value = "60", unitOfMeasurement = "%", deviceClass = "humidity",
      displayMode = "box"
  )
  private ConfigNumber humidityThreshold;

  @State
  @Config(
      friendlyName = "Dauer", icon = "mdi:timer-outline",
      value = "45", unitOfMeasurement = "m", min = 5, max = 120, step = 5
  )
  private ConfigDuration manualDuration;

  @Provided(
      entityId = "${manual_switch}",
      friendlyName = "Lüftung", icon = "mdi:air-filter"
  )
  private ProvidedSwitch manualSwitch;

  @Override
  public void init(SmartBeans sb) {
    controlSwitch.onStateChanged(evt -> syncState());
    syncState();
  }

  private void syncState() {
    if(controlSwitch.getState() != Switch.State.UNAVAILABLE) {
      manualSwitch.setState(controlSwitch.getState());
    }
  }

  @OnStateTrigger(entity = "${humidity_sensor}", notTo = {"unavailable", "unknown"})
  public void humidityChanged(StateEvent evt) {
    if(evt.getNewState().getStateAsInt(0) > humidityThreshold.toInt()) {
      if(automaticMode.isOn()) {
        turnOn("Luftfeuchtigkeit ist " + evt.getNewState().getStateAsInt(0) + "%");
        automatic = true;
      }
    }
    else if(automatic) {
      if (!manual) {
        turnOff("Luftfeuchtigkeit ist " + evt.getNewState().getStateAsInt(0) + "%");
      }
      automatic = false;
    }
  }

  @OnSwitchTurnedOn("manualSwitch")
  public void manuallyTurnedOn() {
    turnOn("Manuell eingeschaltet, Laufzeit: " + manualDuration.toMinutes() + "min");
    manual = true;
    if(manualTimer.isRunning()) {
      manualTimer.cancel();
    }
    manualTimer.start(manualDuration.toDuration());
  }

  @OnSwitchTurnedOff("manualSwitch")
  public void manuallyTurnedOff() {
    turnOff("Manuell ausgeschaltet.");
    manual = false;
    if(manualTimer.isRunning()) {
      manualTimer.cancel();
    }
  }

  @OnTimerElapsed("manualTimer")
  public void manualTimeElapsed() {
    if(!automatic) {
      turnOff("Zeit der manuellen Lüftung ist vorbei");
    }
    manual = false;
  }

  private void turnOn(String reason) {
    if(!controlSwitch.isOn()) {
      sb.log("Lüftung einschalten, " + reason);
      controlSwitch.turnOn();
    }
  }

  private void turnOff(String reason) {
    if(controlSwitch.isOn()) {
      sb.log("Lüftung ausschalten, " + reason);
      controlSwitch.turnOff();
    }
  }
}
````

## Explanations

Some additional hints about the source code:

- **Lines 1–9**: The `@SmartBeanDef` annotation creates the [bean device](../basic-concepts/devices) with a master switch
  to enable or disable the whole bean. It also defines three parameters for all used entities, so the bean can easily be
  reused for another ventilation setup.  
- **Lines 26–43**: The bean defines [configuration entities](../basic-concepts/config) for enabling automatic mode, 
  setting the humidity threshold, and configuring the manual duration.  
- **Lines 45–49**: Creates a [provided switch entity](../basic-concepts/provided) in Home Assistant to control the 
  ventilation via the bean.  
- **Lines 51–61**: Synchronizes the state of the physical switch that controls the ventilation with the virtual bean 
  switch used for logic control.  
- **Lines 63–77**: Implements the automatic mode logic.  
- **Lines 80–104**: Implements the manual mode logic using the switch and a [timer](../basic-concepts/timer).  


## Bean State

When you inspect the full state of the bean entity in Home Assistant, you will notice that the entire bean state is 
exposed as attributes. This means you can easily use them in the UI, for example to create visualizations or additional 
dashboards.

````yaml
friendly_name: SmartBean LueftungSteuerung.Badezimmer Bean
icon: mdi:home-automation
last_deployment: "2025-09-16T13:05:38.805264670Z"
definition:
  name: home.LueftungSteuerung.Badezimmer
  params:
    control_switch: switch.bad_luftung_intern
    manual_switch: switch.bad_luftung
    humidity_sensor: sensor.humidity_bad
  class: home.LueftungSteuerung
automaticMode:
  value: true
humidityThreshold:
  value: 65
manualDuration:
  min: 5
  max: 120
  step: 5
  value: 45
automatic: true
manual: true
manualTimer.interval: 2700
manualTimer.repeats: 1
manualTimer.state: RUNNING
manualTimer.start: "2025-09-19T05:34:54.790350186Z"
manualTimer.end: "2025-09-19T06:19:54.790401806Z"
````

## JUnit Tests

Here a a few examples of JUnit testcases:

````java
@TestDefinition(params = {
    @Param(name = "control_switch", value = "switch.control"),
    @Param(name = "manual_switch", value = "switch.manual"),
    @Param(name = "humidity_sensor", value = "sensor.humidity")
})
public class LueftungSteuerung_Test extends SmartBeansTest<LueftungSteuerung> {

  public LueftungSteuerung_Test() {
    super(LueftungSteuerung.class);
  }

  @Test
  public void testAutomatic() {
    configNumber("humidityThreshold").setCurrentValue(65);
    
    switch_("control").setState(Switch.State.OFF).expectTurnOn();
    triggerBean(LueftungSteuerung::humidityChanged, stateEvent().withNewState("68"));
    assertTrue(currentState().asBool("automatic"));
    assertFalse(currentState().asBool("manual"));

    switch_("control").expectTurnOff();
    triggerBean(LueftungSteuerung::humidityChanged, stateEvent().withNewState("62"));
    assertFalse(currentState().asBool("automatic"));
    assertFalse(currentState().asBool("manual"));
  }

  @Test
  public void testManual() {
    switch_("control").setState(Switch.State.OFF).expectTurnOn();
    triggerBean(LueftungSteuerung::manuallyTurnedOn);
    assertFalse(currentState().asBool("automatic"));
    assertTrue(currentState().asBool("manual"));

    switch_("control").expectTurnOff();
    timer("manualTimer").fastForward();
    assertFalse(currentState().asBool("automatic"));
    assertFalse(currentState().asBool("manual"));
  }
}
````
