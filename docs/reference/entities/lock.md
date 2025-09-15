---
sidebar_position: 20
description: "Entities of domain: lock"
---

# Lock

The `Lock` interface represents lock entities in Home Assistant. When you define a field of type `Lock` in your
SmartBean class and annotate it with `@Entity`, the framework automatically injects an object that lets you control the
corresponding lock entity in Home Assistant. This provides a type-safe way to interact with your smart lock.

````java
@Entity("lock.front_door")
private Lock lock;
````

## State

The lock's state is represented by the `Lock.State` enum, which mirrors the possible 
[states](https://www.home-assistant.io/integrations/lock#the-state-of-a-lock-entity) in Home Assistant: `JAMMED`, 
`OPENING`, `LOCKING`, `OPEN`, `UNLOCKING`, `LOCKED`, `UNLOCKED`, `UNKNOWN` and `UNAVAILABLE`. You can query the lock's
current state using these methods:

| Method               | Description                                                |
|----------------------|------------------------------------------------------------|
| `getState()`         | Get the current state of the lock.                         |
| `getStateAsString()` | Returns the original state from the Home Assistant entity. |

## Attributes

The following attributes of a lock entity can be accessed through simple getter methods:

| HA attribute       | Method              | Description                                           |
|--------------------|---------------------|-------------------------------------------------------|
| `friendly_name`    | `getFriendlyName()` | Friendly name of the entity.                          |
| `icon`             | `getIcon()`         | Icon of the entity.                                   |

You can access any additional attributes that are not directly supported through the `getAttributes()` method.

## Services

The `Lock` interface provides several methods to control your lock through Home Assistant services. Here are the
supported operations:

| HA service    | Method     | Description      |
|---------------|------------|------------------|
| `lock.lock`   | `lock()`   | Locks the lock.  |
| `lock.unlock` | `unlock()` | Unlock the lock. |
| `lock.open`   | `open()`   | Open the lock.   |

## Example

````java
public class ASampleBean implements SmartBean {

  @Entity("lock.front_door")
  private Lock lock;

  public void someBeanMethod() {
    lock.open();
  }
}
````

## Access Entities Programmatically

In addition to the annotation-based approach, you can programmatically access lock entities using the `getLock()` 
method of the `SmartBeans` API. You might prefer this programmatic approach over annotations for example when the entity
ID is dynamically generated through business logic and cannot be determined at compile time.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  public void someBeanMethod() {
    Lock lock = sb.getLock("lock.backdoor");
    if(lock.getState() == Lock.State.UNLOCKED) {
      lock.lock();
    }
  }
}
````

import UncachedNote from './_note_uncached_entities.md';

<UncachedNote />