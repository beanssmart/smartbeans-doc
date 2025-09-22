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
- Built-in testing capabilities
- IDE support with code completion

SmartBeans is designed for Java developers who use Home Assistant as their home automation ecosystem. It is especially 
well-suited for complex automation scenarios and for building reliable, testable automation logic.  

While Home Assistant already offers many excellent ways to create automations, SmartBeans is not meant to compete with 
those options. Instead, it provides a familiar environment for developers who are comfortable with Java and related 
technologies.  

If you are new to Java, SmartBeans may not be the right starting point, since you would need to learn Java first. And if 
you don’t enjoy working with Java, you probably won’t enjoy SmartBeans either. But if you do like Java and feel at home 
with it, chances are you will *love* SmartBeans.

## Getting Started

To start using SmartBeans, you'll need:

- Java Development Kit (JDK) 17 or higher for development and testing
- Home Assistant installation supporting add-ons, for example HA OS.
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

