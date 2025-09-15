# Provided Entities

Provided entities are additional entities created by a SmartBean through the MQTT discovery mechanism and attached to 
the [bean device](../basic-concepts/devices) in Home Assistant. These entities can be used to supply data to Home
Assistant or to trigger actions from it.

:::note
Bean devices require the MQTT integration in Home Assistant. They are only available when you have an MQTT broker
running and properly configured in Home Assistant. The provided entities appear as MQTT entities in Home Assistant and
are automatically created through the MQTT integration's discovery mechanism. Therefore, 
[MQTT discovery](https://www.home-assistant.io/integrations/mqtt/#mqtt-discovery) must be enabled in your installation.
:::

import DocCardList from '@theme/DocCardList';

<DocCardList />