# Config Entities

Config entities are additional entities created by a SmartBean through the MQTT discovery mechanism and attached to 
the [bean device](../basic-concepts/devices) in Home Assistant. These entities allow you to configure and adjust the 
behavior of the bean at runtime. For example, you can define thresholds, timeouts, or light presets.

This section provides a reference for all available config entities. For an overview of the general mechanism, see
the [basic concept of config entities](../basic-concepts/config).

:::note
Bean devices require the MQTT integration in Home Assistant. They are only available when you have an MQTT broker
running and properly configured in Home Assistant. The config entities appear as MQTT entities in Home Assistant and
are automatically created through the MQTT integration's discovery mechanism. Therefore, 
[MQTT discovery](https://www.home-assistant.io/integrations/mqtt/#mqtt-discovery) must be enabled in your installation.
:::

import DocCardList from '@theme/DocCardList';

<DocCardList />