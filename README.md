# kiosk-context-card
A Lovelace card that utilizes browser_mod and displays different UI elements per-device



# Intro
I started into the Smart Home rabbit-hole in 2018 by rewiring my entire house with GE/Jasco switches and SmartThings. 

As one does, you start to bump into the limits of what you can do in a closed ecosystem, so I gradually shifted into a completely open Home Assistant system to control 80+ switches and lights and ~20 smart outlets, along with 18 third-party integrations; now, my smart home controls everything from my Christmas decor to my humidors. 

A big project this year was to integrate a dozen Samsung A9 tablets into my walls for fine-grain control of the house and replicate intercom "drop in" features that I have in my phasing-out Alexas. The problem is that nobody wants to maintain 12 simultaneous frontend dashboards to display context-specific content (e.g. prioritize lights and switches in the room that the tablet-kiosk is in). Thus, the need to develop a "kiosk-contextualized content card" came to be.

# Usage

This assumes you already have browser_mod installed and are using a recent version of HA (tested with 2025.10.1). 

First, you need to add kiosk-context-card.js to your Lovelace dashboard as a resource. Most of this code is to reliably probe browser_mod and provide an interface for the frontend to key off of. 

If the endpoint's browser_mod is mapped, it will display the elements for that map. If an endpoint isn't mapped, it will display what's in "default." If "global" is defined, it will show that content for every endpoint. 

Then, add a card in Lovelace with your browser_mod device mapping to kiosk-context-card and what you want to display per-device. I've included a sample YAML that demonstrates how it might work for your environment. 



```
type: custom:kiosk-context-card
card_mod:
  style: |
    ha-card {
      --mush-spacing: 8px !important;
      --ha-card-padding: 8px !important;
      --ha-card-border-radius: 10px !important;
      --grid-card-gap: 6px !important;
      margin: 0 !important;
      padding: 0 !important;
    }

map:
  # Living Room kiosk
  browser_mod_a1b2c3d4_e5f6a7b8:
    name: 🏠 Living Room
    cards:
      - type: grid
        square: false
        columns: 1
        cards:
          - type: custom:mushroom-light-card
            entity: light.living_room_lamp
            name: Floor Lamp
            use_light_color: true
            show_brightness_control: true
            show_color_control: true
            tap_action:
              action: toggle
          - type: custom:mushroom-light-card
            entity: light.tv_backlight
            name: TV Backlight
            show_brightness_control: true
            tap_action:
              action: toggle
          - type: custom:mushroom-chips-card
            alignment: start
            chips:
              - type: template
                content: Movie
                icon: mdi:filmstrip
                tap_action:
                  action: call-service
                  service: scene.turn_on
                  target:
                    entity_id: scene.living_room_movie
              - type: template
                content: Bright
                icon: mdi:brightness-7
                tap_action:
                  action: call-service
                  service: scene.turn_on
                  target:
                    entity_id: scene.living_room_bright

  # Master Bedroom kiosk
  browser_mod_b2c3d4e5_f6a7b8c9:
    name: 🛏️ Master Bedroom
    cards:
      - type: grid
        square: false
        columns: 3
        cards:
          - type: tile
            entity: switch.master_ceiling_fan
            name: Ceiling Fan
            tap_action:
              action: toggle
          - type: tile
            entity: switch.master_box_fan
            name: Box Fan
            tap_action:
              action: toggle
          - type: custom:mushroom-light-card
            entity: light.master_bedside
            name: Bedside Lamp
            show_brightness_control: true
            show_color_control: true
            tap_action:
              action: toggle

  # Office kiosk
  browser_mod_c3d4e5f6_a7b8c9d0:
    name: 💼 Office
    cards:
      - type: grid
        square: false
        columns: 1
        cards:
          - type: custom:mushroom-light-card
            entity: light.office_desk
            name: Desk Lamp
            show_brightness_control: true
            show_color_control: true
            tap_action:
              action: toggle
          - type: custom:mushroom-chips-card
            alignment: start
            chips:
              - type: template
                content: Focus
                icon: mdi:monitor-eye
                tap_action:
                  action: call-service
                  service: scene.turn_on
                  target:
                    entity_id: scene.office_focus
              - type: template
                content: Relax
                icon: mdi:sofa
                tap_action:
                  action: call-service
                  service: scene.turn_on
                  target:
                    entity_id: scene.office_relax

  # Default view (for unmapped kiosks)
  __default__:
    name: ⚙️ Quick Actions
    cards:
      - type: grid
        square: false
        columns: 3
        cards:
          - type: tile
            entity: switch.garage_lights
            name: Garage
            tap_action:
              action: toggle
          - type: tile
            entity: switch.porch_light
            name: Porch
            tap_action:
              action: toggle
          - type: tile
            entity: light.kitchen_overhead
            name: Kitchen
            tap_action:
              action: toggle

  # Universal section (shown everywhere)
  __universal__:
    name: 🌍 Universal Controls
    cards:
      - type: custom:mushroom-template-card
        primary: Home Status
        secondary: "{{ states('sensor.home_mode') | title }}"
        icon: mdi:home-assistant
        icon_color: blue
      - type: custom:mushroom-template-card
        primary: Alarm
        secondary: "{{ states('alarm_control_panel.home_alarm') }}"
        icon: mdi:shield-home
        icon_color: red

```
