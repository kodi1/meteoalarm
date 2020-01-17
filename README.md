# meteoalarm_m
Official component rely on xml api, which is not reliable in some regions.
This custom component parse meteoalarm page - HTML.
It creates a sensor with is a number of alerts for current and next day.
In sensor attributes of there are arrays with info for each alert.
## Example configuration.yaml
```yaml
sensor:
  - platform: meteoalarm_m
    id: 'BG018-Sofia'
    name: alert
    scan_interval: 3600
```

## notification - automation
```yaml
automation:
  - alias: weather_alert
    trigger:
      - platform: state
        entity_id: sensor.alert
    condition:
      - condition: template
        value_template: '{{trigger.from_state.state != trigger.to_state.state}}'
    action:
      - service: script.all_notify
        data_template:
          tit: 'weather'
          msg: >-
            {%- if not is_state('sensor.alert', '0') -%}
              {%- for s  in ['today', 'tomorrow'] -%}
                {%- set v = state_attr('sensor.w_alert', s) -%}
                {%- if v -%}
                  {% for d  in v -%}
                    {{s}} - {{d['code']}} {{d['event']}} {{d['end']}}
                  {% endfor -%}
                {%- endif -%}
              {% endfor -%}
            {% else %}
              alert expired
            {% endif %}
```
![Screenshot](https://github.com/kodi1/meteoalarm/blob/master/images/notification.png?raw=true)
## lovlace alert - Flex Table Card
```yaml
cards:
  - type: conditional
    conditions:
      - entity: sensor.w_alert
        state_not: '0'
    card:
      type: custom:flex-table-card
      title: today
      entities:
        include: sensor.w_alert
      columns:
        - attr_as_list: today
          modify: x.code.concat(' ', x.event)
          name: event
        - attr_as_list: today
          modify: x.txt
          name: description
        - attr_as_list: today
          modify: x.end
          name: expire

  - type: conditional
    conditions:
      - entity: sensor.w_alert
        state_not: '0'
    card:
      type: custom:flex-table-card
      title: tomorrow
      entities:
        include: sensor.w_alert
      columns:
        - attr_as_list: tomorrow
          modify: x.code.concat(' ', x.event)
          name: event
        - attr_as_list: tomorrow
          modify: x.txt
          name: description
        - attr_as_list: tomorrow
          modify: x.end
          name: expire
```
![Screenshot](https://github.com/kodi1/meteoalarm/blob/master/images/alert.png?raw=true)

[![hacs_badge](https://img.shields.io/badge/HACS-Default-orange.svg?style=for-the-badge)](https://github.com/custom-components/hacs)
