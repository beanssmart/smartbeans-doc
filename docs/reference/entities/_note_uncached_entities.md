:::note
For improved efficiency, it is recommended to cache entity objects as member variables rather than retrieving them repeatedly. 
Since the state and attributes of entity objects are cached internally, no additional backend communication is required for 
multiple state retrievals. Note that initial entity object creation always requires at least one request to the Home 
Assistant backend.
:::
