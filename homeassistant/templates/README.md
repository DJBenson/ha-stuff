# Home Assistant Templates
This directory contains Home Assistant templates in jinja format.

Required Home Assistant 2023.4 or newer.

Ensure you have a folder in your /config folder named custom_templates.

Copy the jinja2 scripts to the custom_templates folder and use the action (formerly service) call homeassistant.reload_custom_templates to refresh the template cache.

To use the templates you need to import the template as follows, using a dummy package called answer_question in a file named tools.jinja

```
{% from 'tools.jinja' import answer_question %}
{{ answer_question('light.kitchen') }}
```

Refer to the readme with each template for usage instructions.

* [Rich Location](./README.md#rich-location)

# Rich Location

This template amalgamates data from various sources (all of which are mandatory to use this template);
* [HACS (The Home Assistant Community Store)](https://hacs.xyz) Required to install the iCloud3 integration below
* [iCloud3 Integration](https://github.com/gcobb321/icloud3_v3)
* [The Home Assistant Companion App](https://companion.home-assistant.io/)
  
```
{% from 'rich-location.jinja' import rich-location %}
{{ rich-location((person_entity, ic3_tracker, app_tracker, section_id, hide_if_home)) }}
```

## Usage:
**person_entity**: the person.name entity of the person to track

**ic3_tracker**: the iCloud3 device_tracker entity of this person's device (i.e. device_tracker.my_icloud3_device)

**app_tracker**: the device_tracker entity from the Companion App of the person to track (i.e. device_tracker.my_ha_app_device)

**section_id**: _1 _or_ 2_ (see below)

**hide_if_home**: _true_ or _false_ - this hides the output of the person is home (as per their person entity)

## section_id:
The template provides two sections which can be used either together or separately, one provides the location and the other provides the travel time from that location to "home".

## Examples:

Todo
