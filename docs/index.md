---
sidebar_position: 1
slug: /
title: Introduction
---

# Welcome to SmartBeans

SmartBeans is a powerful Java-based framework that enables you to write smart home automations for Home Assistant. It
combines the flexibility of Home Assistant with the robustness and type safety of Java, allowing developers to create
reliable and maintainable home automation solutions.

## What is SmartBeans?

SmartBeans provides a seamless integration between Java and Home Assistant, offering:

- Type-safe access to Home Assistant entities
- Easy-to-use automation API
- Full Java ecosystem support
- Built-in testing capabilities
- IDE support with code completion

SmartBeans is designed for Java developers using Home Assistant as their home automation ecosystem. While Home Assistant
offers many excellent ways to create automations, SmartBeans doesn't aim to compete with these options. Instead, it
provides a familiar environment for developers who are comfortable with Java and related technologies.

SmartBeans is perfect for:

- Complex automation scenarios
- Integration of multiple smart home devices
- Building reliable and testable automation logic
- Creating reusable automation components

## Getting Started

To start using SmartBeans, you'll need:

- Java Development Kit (JDK) 17 or higher for development and testing
- Home Assistant installation with installed SmartBeans add-on
- MQTT Home Assistant integration (optional, but required for certain features)
- Basic understanding of Java programming

Check out our [Quick Start Guide](getting-started) to begin your journey with SmartBeans.

## Quick Example

Here's a simple example of a SmartBean that controls a light based on motion detection:

````java
public class MotionLightControl implements SmartBean {

  @Entity("light.kitchen_ceiling")
  private Light light;
  
  @TimerDef(interval = @Interval(seconds = 60))
  private Timer timer;

  @OnStateTrigger(entity = "binary_sensor.motion_kitchen", to = {"on", "off"})
  public void onMotionChange(StateEvent stateEvent) {
    var newState = State.fromValue(stateEvent.getNewState().getState());
    if(newState == State.ON) {
      light.turnOn(brightnessPercent(80), transition(1));
      if(timer.isRunning()) {
        timer.cancel();
      }
    }
    else {
      timer.start();
    }
  }
  
  @OnTimerElapsed(name = "timer")
  public void turnOffLight() {
    light.turnOff(transition(1));
  }
}
````

