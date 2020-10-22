---
layout: default
title: Triggers
parent: Flows
grand_parent : Thingsplex UI
nav_order: 1
---

# Triggers

Trigger is a special node type that is listening for events and starts flow execution.

## Generic trigger

Generic trigger can be configured to react on any event generated either by device or application.

![Small generic node](img/trigger_node_small.png)

Trigger node is configured either using *Normal* or *Advanced* node.

### Normal/basic mode

User configures trigger by selecting objects using dropdown selectors.

![Trigger normal mode](img/trigger_node_exp_1.png)

### Advanced mode

The mode should be used only when device/event can't be configured using Normal mode.The mode imply that user know exact device-service topic address,service name and interface. Address can be dynamically configured using templating syntax, for instance 
```
{% raw %}
pt:j1/mt:evt/rt:dev/rn:zw/ad:1/sv:scene_ctrl/ad:{{setting "dev.address"}}_0
{% endraw %}
```
or 
```
{% raw %}
pt:j1/mt:evt/rt:dev/rn:zw/ad:1/sv:scene_ctrl/ad:{{variable "address" false}}_0
{% endraw %}
```

![Trigger normal mode](img/trigger_node_exp_2.png)

### Virtual devices

A node will be part of virtual device if `Register as virtual service` is selected.

### Message filtering

Trigger node supports optional payload level filtering.

![Trigger normal mode](img/trigger_node_exp_3.png)


## Home event

## Time trigger
