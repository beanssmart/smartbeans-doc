# Entities

Let's explore all entity types that are directly supported by SmartBeans. All these entity objects can be injected into a 
SmartBean class using the `@Entity` annotation. You only need to provide the entity ID of the Home Assistant entity that 
you want to control with this object:

````java
public class ASampleBean implements SmartBean {
  
  @Entity("light.kitchen_ceiling")
  private Light light;
  
}
````

The state of each entity is cached and shared across all beans. When calling methods like `getState()`, the current 
state is only retrieved from Home Assistant if needed - otherwise the cached value is returned. State updates are 
handled automatically through Home Assistant's callback mechanism. This means you don't need to worry about manually 
refreshing states. You will _always_ get the current entity value, while benefiting from getter-like performance due to 
the caching.

import DocCardList from '@theme/DocCardList';

<DocCardList />