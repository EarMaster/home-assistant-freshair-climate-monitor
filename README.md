# Fresh Air Climate Monitor

> **Current Version: 1.0.3**

A comprehensive Home Assistant blueprint that provides intelligent climate control through both active monitoring and proactive recommendations for optimal fresh air management.

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A//raw.githubusercontent.com/EarMaster/home-assistant-freshair-climate-monitor/main/freshair-climate-monitor.yaml)

## 🌟 Features

- **Smart Fresh Air Detection**: Manages ventilation when doors/windows are open
- **Proactive Outdoor Analysis**: Recommends opening when outdoor conditions would help
- **Flexible Threshold Configuration**: Use direct values or input_number helpers for centralized control
- **Multiple Opening Support**: Monitor multiple doors and windows per room
- **Rich Action Context**: Detailed variables for sophisticated automation responses
- **Robust Error Handling**: Graceful handling of sensor unavailability and edge cases

## 🧠 Intelligent Operation Modes

### Combined Mode (Recommended)
The blueprint automatically operates in both modes simultaneously:

**Climate Management** (when openings are open):
- Monitor temperature and humidity thresholds
- Take action when conditions exceed acceptable ranges
- Provide context about current ventilation status

**Opening Recommendations** (when openings are closed):
- Compare indoor vs outdoor conditions
- Suggest opening when outdoor air would improve indoor climate
- Calculate optimal timing for natural climate control

### Individual Mode Control
You can also enable/disable each function independently:
- **Opening Recommendations only**: Smart suggestions when to open windows/doors
- **Closing Recommendations only**: Smart suggestions when to close windows/doors  
- **Both enabled**: Full intelligent climate assistance
- **Neither enabled**: Basic threshold monitoring without recommendations

## 🎯 Trigger IDs and Their Meanings

The blueprint uses specific trigger IDs to identify different climate scenarios. Understanding these helps you create more targeted actions:

### Basic Threshold Triggers
These activate when **both recommendation modes are disabled** for simple monitoring:

| Trigger ID | Condition | When It Fires |
|------------|-----------|---------------|
| `threshold_temp_high` | Temperature above maximum | Simple alert when too hot |
| `threshold_temp_low` | Temperature below minimum | Simple alert when too cold |
| `threshold_humidity_high` | Humidity above maximum | Simple alert when too humid |
| `threshold_humidity_low` | Humidity below minimum | Simple alert when too dry |

### Opening Recommendation Triggers
These activate when **openings are closed** and outdoor conditions could help:

| Trigger ID | Condition | When It Fires |
|------------|-----------|---------------|
| `open_temp_high` | Room too hot, outside cooler | Recommend opening for natural cooling |
| `open_temp_low` | Room too cold, outside warmer | Recommend opening for natural warming |
| `open_humidity_high` | Room too humid, outside drier | Recommend opening for dehumidifying |
| `open_humidity_low` | Room too dry, outside more humid | Recommend opening for humidifying |

### Closing Recommendation Triggers
These activate when **openings are open** but outdoor conditions are worse than indoor:

| Trigger ID | Condition | When It Fires |
|------------|-----------|---------------|
| `close_temp_high` | Room too hot, outside even hotter | Recommend closing for better cooling |
| `close_temp_low` | Room too cold, outside even colder | Recommend closing for better warming |
| `close_humidity_high` | Room too humid, outside even more humid | Recommend closing for better dehumidifying |
| `close_humidity_low` | Room too dry, outside even drier | Recommend closing for better humidifying |

### Quick Reference
```yaml
# In your actions, use trigger.id to determine the scenario:
- condition: template
  value_template: "{{ trigger.id.startswith('threshold_') }}"  # Basic threshold alerts
- condition: template
  value_template: "{{ trigger.id.startswith('open_') }}"  # Any opening recommendation
- condition: template
  value_template: "{{ trigger.id.startswith('close_') }}"  # Any closing recommendation
- condition: template
  value_template: "{{ trigger.id == 'open_temp_high' }}"  # Specific cooling opportunity
- condition: template
  value_template: "{{ 'temp' in trigger.id }}"  # Any temperature-related trigger
- condition: template
  value_template: "{{ 'humidity' in trigger.id }}"  # Any humidity-related trigger
```

## 📱 Enhanced Example Actions

### Comprehensive Notification System

```
action:
  - choose:
      # Basic threshold alerts (simple monitoring mode)
      - conditions:
          - condition: template
            value_template: "{{ trigger.id.startswith('threshold_') }}"
        sequence:
          - service: notify.mobile_app_your_phone
            data:
              title: "🌡️ Climate Alert: {{ room_name }}"
              message: >
                {% if trigger.id == 'threshold_temp_high' %}
                  🔥 Temperature is high: {{ current_temperature }}°C (Max: {{ temp_max }}°C)
                  💡 Consider your cooling options or check ventilation
                {% elif trigger.id == 'threshold_temp_low' %}
                  🧊 Temperature is low: {{ current_temperature }}°C (Min: {{ temp_min }}°C)
                  💡 Consider your heating options or close windows
                {% elif trigger.id == 'threshold_humidity_high' %}
                  💧 Humidity is high: {{ current_humidity }}% (Max: {{ humidity_max }}%)
                  💡 Consider dehumidification or improving air circulation
                {% elif trigger.id == 'threshold_humidity_low' %}
                  🏜️ Humidity is low: {{ current_humidity }}% (Min: {{ humidity_min }}%)
                  💡 Consider humidification or reducing air circulation
                {% endif %}

      # Opening recommendations when all doors/windows closed
      - conditions:
          - condition: template
            value_template: "{{ trigger.id.startswith('open_') }}"
        sequence:
          - service: notify.mobile_app_your_phone
            data:
              title: "💨 Fresh Air Opportunity: {{ room_name }}"
              message: >
                {% if trigger.id == 'open_temp_high' %}
                  🌡️ Time to open windows for natural cooling!
                  Outside: {{ outdoor_temperature }}°C ({{ temperature_difference | abs }}°C cooler)
                  Inside: {{ current_temperature }}°C ({{ temp_max }}°C max)
                  💡 Opening windows could naturally cool your room!
                {% elif trigger.id == 'open_temp_low' %}
                  ☀️ Great time to open windows for natural warming!
                  Outside: {{ outdoor_temperature }}°C ({{ temperature_difference }}°C warmer)
                  Inside: {{ current_temperature }}°C ({{ temp_min }}°C min)
                  💡 Let the sunshine help warm your space!
                {% elif trigger.id == 'open_humidity_high' %}
                  💧 Open windows to reduce humidity naturally!
                  Outside: {{ outdoor_humidity }}% ({{ humidity_difference | abs }}% drier)
                  Inside: {{ current_humidity }}% ({{ humidity_max }}% max)
                  💡 Fresh air circulation can help reduce moisture!
                {% elif trigger.id == 'open_humidity_low' %}
                  🌿 Outdoor air can help increase indoor humidity!
                  Outside: {{ outdoor_humidity }}% ({{ humidity_difference }}% more humid)
                  Inside: {{ current_humidity }}% ({{ humidity_min }}% min)
                  💡 Natural moisture from outside air could help!
                {% endif %}
              data:
                actions:
                  - action: "REMIND_LATER"
                    title: "Remind in 30 min"
                  - action: "DISMISS_RECOMMENDATION"
                    title: "Not suitable now"
                  - action: "VIEW_DETAILS"
                    title: "More info"

      # Climate management when doors/windows are open
      - conditions:
          - condition: template
            value_template: "{{ trigger.id.startswith('close_') }}"
        sequence:
          - service: notify.mobile_app_your_phone
            data:
              title: "🌡️ Climate Alert: {{ room_name }}"
              message: >
                {% if trigger.id == 'close_temp_high' %}
                  🔥 Temperature is high and outside is even hotter
                  Current: {{ current_temperature }}°C (Max: {{ temp_max }}°C)
                  Outside: {{ outdoor_temperature }}°C
                  💡 Consider closing windows to prevent more heat from entering
                {% elif trigger.id == 'close_temp_low' %}
                  🧊 Temperature is low and outside is even colder
                  Current: {{ current_temperature }}°C (Min: {{ temp_min }}°C)
                  Outside: {{ outdoor_temperature }}°C
                  💡 Consider closing windows to prevent more cold air from entering
                {% elif trigger.id == 'close_humidity_high' %}
                  💧 High humidity and outside air is even more humid
                  Current: {{ current_humidity }}% (Max: {{ humidity_max }}%)
                  Outside: {{ outdoor_humidity }}%
                  💡 Consider closing windows to prevent more moisture from entering
                {% elif trigger.id == 'close_humidity_low' %}
                  🏜️ Low humidity and outside air is even drier
                  Current: {{ current_humidity }}% (Min: {{ humidity_min }}%)
                  Outside: {{ outdoor_humidity }}%
                  💡 Consider closing windows to prevent more dry air from entering
                {% endif %}
              data:
                actions:
                  - action: "CLIMATE_ADJUST"
                    title: "Suggest adjustments"
                  - action: "VIEW_SENSORS"
                    title: "Check all sensors"
```

### Smart TTS Announcements

```
action:
  - service: tts.speak
    data:
      entity_id: media_player.{{ room_name | lower | replace(' ', '_') }}_speaker
      message: >
        {% if trigger.id.startswith('open_') %}
          Climate tip: It's {{ outdoor_temperature }} degrees outside, 
          {% if trigger.id == 'open_temp_high' %}
            {{ temperature_difference | abs }} degrees cooler than your {{ room_name }}.
            Opening your {{ door_window_sensors | count }} window{% if door_window_sensors | count > 1 %}s{% endif %} 
            could naturally cool the room from {{ current_temperature }} to closer to your 
            {{ temp_max }} degree maximum.
          {% elif trigger.id == 'open_humidity_high' %}
            with {{ humidity_difference | abs }} percent lower humidity. 
            Opening windows could help reduce your indoor humidity from {{ current_humidity }} percent 
            toward your {{ humidity_max }} percent target.
          {% endif %}
        {% else %}
          Attention: Your {{ room_name }} {{ trigger.id | replace('_', ' ') }} threshold has been reached.
          Current conditions: {{ current_value }}{{ unit }}, with {{ open_sensors_count }} opening providing fresh air.
          {% if trigger.id in ['close_temp_high', 'close_humidity_high'] %}
            Outside conditions are worse. Consider closing windows.
          {% else %}
            Outside conditions are worse. Consider closing windows.
          {% endif %}
        {% endif %}
```

### Adaptive Lighting Integration

```
action:
  - choose:
      # Cooling recommendation - suggest dimming lights
      - conditions:
          - condition: template
            value_template: "{{ trigger.id == 'open_temp_high' }}"
        sequence:
          - service: light.turn_on
            target:
              entity_id: light.{{ room_name | lower | replace(' ', '_') }}_main
            data:
              brightness_pct: 30
              color_name: "blue"
          - delay:
              seconds: 3
          - service: light.turn_off
            target:
              entity_id: light.{{ room_name | lower | replace(' ', '_') }}_main

      # Warming recommendation - suggest bright warm lights
      - conditions:
          - condition: template
            value_template: "{{ trigger.id == 'open_temp_low' }}"
        sequence:
          - service: light.turn_on
            target:
              entity_id: light.{{ room_name | lower | replace(' ', '_') }}_main
            data:
              brightness_pct: 80
              color_temp: 2700
```

### Dashboard Integration with Actionable Cards

```
action:
  - service: input_text.set_value
    target:
      entity_id: input_text.climate_recommendations
    data:
      value: >
        {% if trigger.id.startswith('open_') %}
          {{ room_name }}: {{ trigger.id | replace('_', ' ') | title }} opportunity! 
          Outside {{ outdoor_temperature }}°C vs inside {{ current_temperature }}°C
        {% else %}
          {{ room_name }}: {{ trigger.id | replace('_', ' ') | title }} alert - {{ current_value }}{{ unit }}
        {% endif %}
        
  - service: persistent_notification.create
    data:
      title: "Fresh Air Climate Monitor"
      message: >
        Room: {{ room_name }}
        Action: {{ trigger.id | replace('_', ' ') | title }}
        {% if trigger.id.startswith('open_') %}
        Recommendation: Open windows/doors
        Outdoor: {{ outdoor_temperature }}°C / {{ outdoor_humidity }}%
        {% else %}
        Recommendation: Close windows/doors
        {% endif %}
        Indoor: {{ current_temperature }}°C / {{ current_humidity }}%
        Openings: {{ open_sensors_count if trigger.id.startswith('close_') else 'All closed' }}
      notification_id: "fresh_air_{{ room_name | lower | replace(' ', '_') }}"
```

## 🔄 Advanced Integration Examples

### Smart Home Ecosystem Integration

```
# Integration with HVAC systems
action:
  - choose:
      - conditions:
          - condition: template
            value_template: "{{ trigger.id == 'open_temp_high' and temperature_difference | abs > 5 }}"
        sequence:
          - service: climate.turn_off
            target:
              entity_id: climate.{{ room_name | lower | replace(' ', '_') }}
          - service: notify.mobile_app_your_phone
            data:
              message: "HVAC turned off - natural cooling available via windows!"

      - conditions:
          - condition: template
            value_template: "{{ trigger.id == 'close_temp_high' and open_sensors_count > 0 }}"
        sequence:
          - service: fan.turn_on
            target:
              entity_id: fan.{{ room_name | lower | replace(' ', '_') }}_ceiling
          - service: fan.set_percentage
            target:
              entity_id: fan.{{ room_name | lower | replace(' ', '_') }}_ceiling
            data:
              percentage: "{{ ((current_temperature - temp_max) * 20) | int | abs | min(100) }}"
```

This combined approach provides the most comprehensive climate intelligence, automatically adapting to your home's current state while proactively suggesting improvements. The rich example actions demonstrate how to create sophisticated responses that integrate with your entire smart home ecosystem.