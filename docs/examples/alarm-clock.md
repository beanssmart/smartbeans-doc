---
sidebar_position: 30
description: "Alarm clock with snooze functionality and advanced wake-up experience controlling light and music."
title: Alarm Clock
---

# Smart Alarm Clock

This example demonstrates a **smart alarm clock** implemented with SmartBeans. It coordinates light, music, and routines 
in Home Assistant to create a comfortable wake-up experience. The alarm can be configured to wake you up with light, 
with music, or both - and includes features like **snooze**, **automatic routines**, and **pre-wake lighting**.

## Overview

The `Wecker` (german word for _alarm clock_) bean  controls the wake-up process using [time triggers](../reference/trigger/time), 
[timers](../basic-concepts/timer), and [configuration entities](../basic-concepts/config). It listens for a defined wake-up
time, starts a pre-wake phase (light gradually turning on), and then starts the alarm (music). It also supports a **snooze** 
button and launches a "Good Morning" routine afterward.

The wake-up logic is flexible and configurable directly in Home Assistant, with options to enable/disable light or music
wake-ups and to activate or deactivate the routine afterward.

## Behavior Summary

| Situation              | Action                                 |
|------------------------|----------------------------------------|
| 30 min before wake-up  | Light fades in gradually               |
| At wake-up time        | Music starts playing                   |
| After wake-up          | Optional “Good Morning” routine starts |
| Snooze pressed         | Music or light pauses temporarily      |
| User leaves bed        | All actions stop, automation restored  |

## Key Components

### Bean and Device Definition

````java
@SmartBeanDef(beanDevice = @BeanDevice(type = BeanDevice.EntityType.SWITCH), params = {
    @Param(name = "entity.weckzeit", value = "input_datetime.weckzeit")
})
public class Wecker implements SmartBean {

  private SmartBeans sb;

}
````

The annotation defines the bean device and links the wake-up time (`input_datetime.weckzeit`) from Home Assistant as a 
configurable [parameter](../basic-concepts/instances#bean-parameters). The bean itself appears as a switch entity in Home
Assistant, which can be used to enable or disable the entire bean - and therefore the alarm clock itself.

The field `sb` is injected at runtime with the `SmartBeans` instance and provides access to the SmartBeans framework API.  

### Provided Entities

````java
  @Provided(entityId = "button.snooze_wecker", friendlyName = "Snooze", icon = "mdi:alarm-snooze")
  private ProvidedButton snooze;
````

A [provided button](../reference/provided/button) named _Snooze_ is created in Home Assistant. When pressed, it calls
the `snoozeWecker()` method we implement later to handle snooze behavior.

### Configurable Options

````java
  @State
  @Config(value = "true", friendlyName = "Wecker Musik", icon = "mdi:bell-outline")
  private ConfigBoolean weckeMitMusik;

  @State
  @Config(value = "true", friendlyName = "Wecker Licht", icon = "mdi:alarm-light-outline")
  private ConfigBoolean weckeMitLicht;

  @State
  @Config(value = "false", friendlyName = "Routine starten", icon = "mdi:weather-sunset-up")
  private ConfigBoolean routineStarten;
````

These [configuration entities](../basic-concepts/config) allow you to toggle:
- Whether to wake up with music
- Whether to wake up with light
- Whether to start a “Good Morning” routine after waking up

Each option appears as a Home Assistant switch that can be adjusted directly in the UI.

### Bean State and Timers 

````java
  @State
  private boolean lichtAn;
  @State
  private boolean musikPlaying;

  @Entity("light.lightstrip_bett")
  private Light licht;

  @State
  @TimerDef(interval = @Interval(minutes = 30))
  private Timer klingelTimer;

  @State
  @TimerDef(interval = @Interval(minutes = 5))
  private Timer routineTimer;

  @State
  @TimerDef(interval = @Interval(minutes = 5))
  private Timer snoozeTimer;
````

The `boolean` values `lichtAn` and `musikPlaying` indicate whether the bean is currently controlling the lighting or the 
music in the bedroom.

`licht` is the light entity controlled by the alarm clock. It will be used later in the bean to manage lighting behavior 
as part of the wake-up process.

Three [timers](../basic-concepts/timer) manage the different wake-up phases:

| Timer Name     | Interval   | Purpose                                                           |
|----------------|------------|-------------------------------------------------------------------|
| `klingelTimer` | 30 minutes | Triggers the main wake-up (music, etc.) after the pre-light phase |
| `routineTimer` | 5 minutes  | Waits before starting the post-alarm morning routine              |
| `snoozeTimer`  | 5 minutes  | Controls snooze duration before restarting the alarm              |

### Wake-Up Flow

#### Step 1: Pre-Wake Phase (Light Gradually Turns On)

````java
  @OnTimeTrigger(entityId = "${entity.weckzeit}", offset = "-00:30:00")
  public void startAufwecken() {
    if(weckeMitLicht.isOn()) {
      lichtAn = true;
      sb.log("Aufwecken wird gestartet, Licht wird langsam eingeschaltet.");
      //Automatische Steuerung deaktivieren
      sb.getInputBoolean("input_boolean.bewegung_steuerung_schlafzimmer").turnOff();
      licht.turnOn(brightness(255), rgbColor(255, 255, 255), transition(Duration.ofMinutes(30)));
    }
    else {
      sb.log("Aufwecken wird gestartet, wecken mit Licht ist ausgeschaltet.");
    }
    klingelTimer.start();
  }
````

This trigger starts 30 minutes before the configured wake-up time. If "wake with light" is enabled, it smoothly turns on
the bed light over 30 minutes. Motion-based automation is temporarily disabled to prevent interference.

#### Step 2: Alarm Starts

````java
  @OnTimerElapsed("klingelTimer")
  @OnTimerElapsed("snoozeTimer")
  public void klingeln() {
    if(weckeMitMusik.isOn()) {
      sb.log("Der Wecker klingelt.");
      musikPlaying = true;
      sb.getBean(SonosBox.class, "Schlafzimmer").invoke(box -> box.playSource(MusicSource.NJOY));
    }
    else {
      sb.log("Der Wecker klingelt nicht, wecken mit Musik ist ausgeschaltet.");
    }
    routineTimer.start();
  }
````

When the timer elapses, the actual alarm starts. If “wake with music” is enabled, it plays a radio station (e.g., _NJOY_)
via a `SonosBox` SmartBean.

#### Step 3: Optional Routine

````java
  @OnTimerElapsed("routineTimer")
  public void startRoutine() {
    lichtAn = false;
    musikPlaying = false;
    if(routineStarten.isOn()) {
      sb.log("Wecker beendet, Routine 'Guten Morgen' starten.");
      sb.getBean(Routinen.class).invoke(Routinen::startGutenMorgen);
    }
    if(weckeMitLicht.isOn()) {
      licht.turnOff(transition(Duration.ofSeconds(30)));
      //Automatische Steuerung wieder einschalten
      sb.getInputBoolean("input_boolean.bewegung_steuerung_schlafzimmer").turnOn();
    }
  }
````

After the alarm, if the "start routine" option is enabled, it triggers a _Good Morning_ routine through another SmartBean 
(`Routinen`). Lights are turned off with a smooth transition.

### Snooze Logic

````java
  @OnButtonPressed("snooze")
  public void snoozeWecker() {
    if(musikPlaying) {
      sb.log("Wecker snoozen (real snooze).");
      musikPlaying = false;
      sb.getBean(SonosBox.class, "Schlafzimmer").invoke(box -> box.pause());
      routineTimer.cancel();
      snoozeTimer.start();
    }
    else if(lichtAn) {
      sb.log("Wecker snoozen (pre snooze).");
      lichtAn = false;
      licht.turnOff(transition(Duration.ofSeconds(5)));
      //Automatische Steuerung wieder einschalten
      sb.getInputBoolean("input_boolean.bewegung_steuerung_schlafzimmer").turnOn();
    }
  }
````

The Snooze button behaves contextually:
- If music is playing, it stops playback, cancels the routine timer, and starts a 5-minute snooze timer.
- If only light is on, it turns off the light gently (a “pre-snooze” mode).

### Cancelling the Alarm

````java
  @OnStateTrigger(entity = "input_boolean.sleeping", to = "off")
  public void cancelWecker() {
    sb.log("Wecker abgebrochen.");
    klingelTimer.cancel();
    routineTimer.cancel();
    snoozeTimer.cancel();
    if(lichtAn) {
      licht.turnOff(transition(Duration.ofSeconds(5)));
      //Automatische Steuerung wieder einschalten
      sb.getInputBoolean("input_boolean.bewegung_steuerung_schlafzimmer").turnOn();
    }
    lichtAn = false;
    musikPlaying = false;
  }
````

When the user gets out of bed (`input_boolean.sleeping` turns off), all timers are cancelled, music and lights are
stopped, and the room’s automation is re-enabled.

## Example Automation Flow

1. User sets input_datetime.weckzeit to 07:00.
2. At 06:30, the light begins to fade in gradually.
3. At 07:00, music starts playing.
4. After 5 minutes, the morning routine begins.
5. If Snooze is pressed, music stops for 5 minutes before resuming.
6. When the user gets out of bed, all wake-up actions stop automatically.

## Bean Device in Home Assistant

This is a screenshot of all entities of the [bean device](../basic-concepts/devices) in Home Assistant.

![Bean device in Home Assistant](/img/examples/alarm_clock_device.png)

## JUnit Tests

Here a a few examples of JUnit testcases:

````java
public class Wecker_Test extends SmartBeansTest<Wecker> {
  public Wecker_Test() {
    super(Wecker.class);
  }

  @Test
  public void testStartAufweckenMitLicht() {
    configBoolean("weckeMitLicht").setCurrentValue(true);
    inputBoolean("input_boolean.bewegung_steuerung_schlafzimmer").expectTurnOff();
    light("light.lightstrip_bett_intern").expectTurnOn(atLeast(transition(Duration.ofMinutes(30))));
    triggerBean(Wecker::startAufwecken);
    expectRunningTimer("klingelTimer");
  }

  @Test
  public void testStartAufweckenOhneLicht() {
    configBoolean("weckeMitLicht").setCurrentValue(false);
    triggerBean(Wecker::startAufwecken);
    expectRunningTimer("klingelTimer");
  }

  @Test
  public void testKlingelnMitMusik() {
    configBoolean("weckeMitMusik").setCurrentValue(true);

    List<MusicSource> playedSources = new ArrayList<>();
    bean(SonosBox.class, "Schlafzimmer").expectInvocation(new SonosBox() {
      @Override
      public void playSource(MusicSource source) {
        playedSources.add(source);
      }
    });
    triggerBean(Wecker::klingeln);

    assertEquals(1, playedSources.size());
    assertEquals(MusicSource.NJOY, playedSources.get(0));

    expectRunningTimer("routineTimer");
  }

  @Test
  public void testFullRunWithSnooze() {
    configBoolean("weckeMitLicht").setCurrentValue(true);
    configBoolean("weckeMitMusik").setCurrentValue(true);
    configBoolean("routineStarten").setCurrentValue(true);

    //Licht
    inputBoolean("input_boolean.bewegung_steuerung_schlafzimmer").expectTurnOff();
    light("light.lightstrip_bett_intern").expectTurnOn(atLeast());
    triggerBean(Wecker::startAufwecken);

    //Musik
    bean(SonosBox.class, "Schlafzimmer").expectInvocation();
    nextElapsedTimer().get().fastForward();

    //Snooze
    bean(SonosBox.class, "Schlafzimmer").expectInvocation();
    triggerBean(Wecker::snoozeWecker);

    //Wieder Musik
    bean(SonosBox.class, "Schlafzimmer").expectInvocation();
    nextElapsedTimer().get().fastForward();

    //Routine
    bean(Routinen.class).expectInvocation();
    inputBoolean("input_boolean.bewegung_steuerung_schlafzimmer").expectTurnOn();
    light("light.lightstrip_bett_intern").expectTurnOff(atLeast());
    nextElapsedTimer().get().fastForward();
  }
}
````