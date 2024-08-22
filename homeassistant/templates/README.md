# Home Assistant Templates
This directory contains Home Assistant templates in jinja2 format.

Required Home Assistant 2023.4 or newer.

Ensure you have a folder in your /config folder named custom_templates.

Copy the jinja2 scripts to the custom_templates folder and use the action (formerly service) call homeassistant.reload_custom_templates to refresh the template cache.

To use the templates you need to import the template as follows, using a dummy package called answer_question in a file named tools.jinja

```
{% from 'tools.jinja' import answer_question %}
{{ answer_question('light.kitchen') }}
```

Refer to the readme with each template for usage instructions.
