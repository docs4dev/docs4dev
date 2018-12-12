## 43. Addressing All Instances of a Service

The “destination” parameter is used in a Spring  `PathMatcher`  (with the path separator as a colon —  `:` ) to determine if an instance processes the message. Using the example from earlier,  `/bus-env/customers:**`  targets all instances of the “customers” service regardless of the rest of the service ID.
