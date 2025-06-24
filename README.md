# Fresh Air Climate Monitor

> **Current Version: 1.0.0**

A comprehensive Home Assistant blueprint that provides intelligent climate control through both active monitoring and proactive recommendations for optimal fresh air management.

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A//raw.githubusercontent.com/EarMaster/home-assistant-freshair-climate-monitor/main/freshair-climate-monitor.yaml)

## 🌟 Features

- **Dual Intelligence System**: Combines climate management with opening recommendations
- **Smart Fresh Air Detection**: Manages climate when doors/windows are open
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
- Climate Management only: Traditional threshold monitoring
- Recommendations only: Pure advisory system
- Both enabled: Full intelligent climate assistance

## 📱 Enhanced Example Actions

### Comprehensive Notification System

```
action:
  - choose:
      # Opening recommendations when all doors/windows closed
      - conditions:
          - condition: template
            value_template: "{{ 'recommend' in trigger.id }}"
        sequence:
          - service: notify.mobile_app_your_phone
            data:
              title: "💨 Fresh Air Opportunity: {{ room_name }}"
              message: >
                {% if recommendation_type == 'cooling' %}
                  🌡️ Time to open windows for natural cooling!
                  Outside: {{ outdoor_temperature }}°C ({{ temperature_difference | abs }}°C cooler)
                  Inside: {{ current_temperature }}°C ({{ temp_max }}°C max)
                  💡 Opening windows could naturally cool your room!
                {% elif recommendation_type == 'warming' %}
                  ☀️ Great time to open windows for natural warming!
                  Outside: {{ outdoor_temperature }}°C ({{ temperature_difference }}°C warmer)
                  Inside: {{ current_temperature }}°C ({{ temp_min }}°C min)
                  💡 Let the sunshine help warm your space!
                {% elif recommendation_type == 'dehumidify' %}
                  💧 Open windows to reduce humidity naturally!
                  Outside: {{ outdoor_humidity }}% ({{ humidity_difference | abs }}% drier)
                  Inside: {{ current_humidity }}% ({{ humidity_max }}% max)
                  💡 Fresh air circulation can help reduce moisture!
                {% elif recommendation_type == 'humidify' %}
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
            value_template: "{{ recommendation_type == 'management' }}"
        sequence:
          - service: notify.mobile_app_your_phone
            data:
              title: "🌡️ Climate Alert: {{ room_name }}"
              message: >
                {% if trigger_type == 'temp_high' %}
                  🔥 Temperature is high with {{ open_sensors_count }} opening(s) providing fresh air
                  Current: {{ current_temperature }}°C (Max: {{ temp_max }}°C)
                  💡 Consider adjusting window positions or adding cross-ventilation
                {% elif trigger_type == 'temp_low' %}
                  🧊 Temperature is low despite {{ open_sensors_count }} opening(s)
                  Current: {{ current_temperature }}°C (Min: {{ temp_min }}°C)
                  💡 You might want to close some windows or add heating
                {% elif trigger_type == 'humidity_high' %}
                  💧 High humidity with {{ open_sensors_count }} opening(s) for ventilation
                  Current: {{ current_humidity }}% (Max: {{ humidity_max }}%)
                  💡 Increase air circulation or consider mechanical dehumidification
                {% elif trigger_type == 'humidity_low' %}
                  🏜️ Low humidity despite fresh air access
                  Current: {{ current_humidity }}% (Min: {{ humidity_min }}%)
                  💡 Add moisture sources or reduce ventilation temporarily
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
        {% if 'recommend' in trigger.id %}
          Climate tip: It's {{ outdoor_temperature }} degrees outside, 
          {% if recommendation_type == 'cooling' %}
            {{ temperature_difference | abs }} degrees cooler than your {{ room_name }}.
            Opening your {{ door_window_sensors | count }} window{% if door_window_sensors | count > 1 %}s{% endif %} 
            could naturally cool the room from {{ current_temperature }} to closer to your 
            {{ temp_max }} degree maximum.
          {% elif recommendation_type == 'dehumidify' %}
            with {{ humidity_difference | abs }} percent lower humidity. 
            Opening windows could help reduce your indoor humidity from {{ current_humidity }} percent 
            toward your {{ humidity_max }} percent target.
          {% endif %}
        {% else %}
          Attention: Your {{ room_name }} {{ trigger_type | replace('_', ' ') }} threshold has been reached.
          Current conditions: {{ current_value }}{{ unit }}, with {{ open_sensors_count }} opening providing fresh air.
          {% if trigger_type in ['temp_high', 'humidity_high'] %}
            Consider increasing ventilation.
          {% else %}
            You may want to reduce air circulation.
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
            value_template: "{{ recommendation_type == 'cooling' }}"
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
            value_template: "{{ recommendation_type == 'warming' }}"
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
        {% if 'recommend' in trigger.id %}
          {{ room_name }}: {{ recommendation_type | title }} opportunity! 
          Outside {{ outdoor_temperature }}°C vs inside {{ current_temperature }}°C
        {% else %}
          {{ room_name }}: {{ trigger_type | replace('_', ' ') | title }} alert - {{ current_value }}{{ unit }}
        {% endif %}
        
  - service: persistent_notification.create
    data:
      title: "Fresh Air Climate Monitor"
      message: >
        Room: {{ room_name }}
        Action: {{ recommendation_type | title if 'recommend' in trigger.id else 'Climate Management' }}
        {% if 'recommend' in trigger.id %}
        Recommendation: {{ recommendation_type | title }}
        Outdoor: {{ outdoor_temperature }}°C / {{ outdoor_humidity }}%
        {% endif %}
        Indoor: {{ current_temperature }}°C / {{ current_humidity }}%
        Openings: {{ open_sensors_count if recommendation_type == 'management' else 'All closed' }}
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
            value_template: "{{ recommendation_type == 'cooling' and temperature_difference | abs > 5 }}"
        sequence:
          - service: climate.turn_off
            target:
              entity_id: climate.{{ room_name | lower | replace(' ', '_') }}
          - service: notify.mobile_app_your_phone
            data:
              message: "HVAC turned off - natural cooling available via windows!"

      - conditions:
          - condition: template
            value_template: "{{ trigger_type == 'temp_high' and open_sensors_count > 0 }}"
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