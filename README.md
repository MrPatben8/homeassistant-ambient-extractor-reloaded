
# Ambient Extractor Reloaded

Functionality forked from [color_extractor](https://www.home-assistant.io/integrations/color_extractor/), with vastly improved color extraction, ideal for mood lighting.
HACS integration forked from [ambient_extractor](https://github.com/xplus2/homeassistant-ambient-extractor).

## What is AER?
Like color_extractor, this integration will extract the predominant color from a given image and apply it to a target light. 
This version improves the color extraction for use in mood lighting by prioritizing colors over grayscale in the image.
This is achieved by using ColorThief to extract a color palette from the image, then performing a weighted average on the colors, using the color's hue as a weight. This means that grayscale colors and light colors have a lesser impact on the final extracted color, making for more colorful and meaningful lighting. Additionally, this integration also adjusts the light brightness in order to more accurately represent the extracted color.

![Comparison of old color extraction method vs new one](https://i.ibb.co/hxqR7JpX/Comparisson.png)

### Actions
| Key| Example| Description |
|--|--|--|
| ``color_extract_url``| ```https://example.com/images/logo.png```|The full URL (including schema, `http://`, `https://`) of the image to process|
|``color_extract_path``|```/tmp/album.png```|The full path to the image file on local storage we’ll process|
|``entity_id``|``light.shelf_leds``| The RGB capable light we’ll set the color and brightness of


**Please ensure any [external URLs](https://www.home-assistant.io/docs/configuration/basic/#allowlist_external_urls) or [external files](https://www.home-assistant.io/docs/configuration/basic/#allowlist_external_dirs) are authorized for use, you will receive error messages if this component is not allowed access to these external resources.**

Example in `configuration.yaml`:
```
homeassistant:
  allowlist_external_urls:
    - http://yourdevice:port/screenshot
    - http://enigmareceiver/grab?format=png&mode=video&r=64
```
### URL Action

Add the parameter key `color_extract_url` to the action.

This action allows you to pass in the URL of an image, have it downloaded, get the predominant color from it, and then set a light’s RGB value to it.

### File Action

Add the parameter key `color_extract_path` to the action.

This action is very similar to the URL action above, except it processes a file from the local file storage.

## Example Automations

Example usage in an automation, taking the album art present on a Chromecast and supplying it to `light.shelf_leds` whenever it changes:

```yaml
#automation.yaml
- alias: "Chromecast to Shelf Lights"

  triggers:
    - trigger: state
      entity_id: media_player.chromecast

  actions:
    - action: ambient_extractor_reloaded.turn_on
      data_template:
        color_extract_url: "{{ states.media_player.chromecast.attributes.entity_picture }}"
        entity_id: light.shelf_leds
```

With a nicer transition period of 5 seconds and setting brightness to 100% each time (part of the [`light.turn_on`](https://www.home-assistant.io/integrations/light#action-lightturn_on) action parameters):

```yaml
#automation.yaml
- alias: "Nicer Chromecast to Shelf Lights"

  triggers:
    - trigger: state
      entity_id: media_player.chromecast

  actions:
    - action: ambient_extractor_reloaded.turn_on
      data_template:
        color_extract_url: "{{ states.media_player.chromecast.attributes.entity_picture }}"
        entity_id: light.shelf_leds
        brightness_pct: 100
        transition: 5
```




## Installation

### Using HACS

1. Ensure that [HACS](https://github.com/hacs/integration) is installed.
2. Add Custom repository `https://github.com/MrPatben8/homeassistant-ambient-extractor-reloaded.git`. Category `Integration`.
3. Install the "Ambient Extractor Reloaded" integration.
4. Restart Home Assistant.

### Manual installation

1. Copy the folder `ambient_extractor_reloaded` to `custom_components` in your Home Assistant `config` folder.
2. [Configure the integration](#configuration).
3. Restart Home Assistant.

## Configuration
Add the following line to your `configuration.yaml` (not needed when using HACS):

    ambient_extractor_reloaded:


## Usage examples

```yaml
service: ambient_extractor_reloaded.turn_on
data_template:
  ambient_extract_url: "http://enigma2/grab?format=png&mode=video&r=96"
  entity_id:
    - light.living_room_zha_group_0x0002
  transition: 0.4
  
  # bool, default: false
  brightness_auto: true
  
  # string(mean|rms|natural), default: mean
  brightness_mode: natural
  
  # 0-255, default: 2
  brightness_min: 2
  
  # 0-255, default: 70
  brightness_max: 70
```

### Using helper variables

```yaml
service: ambient_extractor_reloaded.turn_on
data_template:
  ambient_extract_url: "http://127.0.0.1:8123{{ states.media_player.firetv.attributes.entity_picture }}"
  entity_id:
    - light.living_room_zha_group_0x0002
  transition: 0.3
  color_temperature: "{{ states('input_number.ambilight_color_temperature') }}"
  brightness_auto: true
  brightness_mode: natural
  brightness_min: "{{ states('input_number.ambilight_brightness_min') }}"
  brightness_max: "{{ states('input_number.ambilight_brightness_max') }}"
```
Create `ambilight_color_temperature` as Number from 1.000 to 40.000, step size 1.

Make sure that `allowlist_external_urls` contains `http://127.0.0.1:8123` when using the `entity_picture` attribute.


### Full automation YAML


#### Using a fast image source

Two times per second, if screenshots can be accessed fast enough. Tested with OpenATV on Vu+ Uno 4K SE.

```yaml
alias: Ambient Light enigma2
description: ""
trigger:
  - platform: time_pattern
    seconds: "*"
    minutes: "*"
    hours: "*"
condition:
  - condition: state
    entity_id: media_player.enigma2
    state: playing
action:
  - service: ambient_extractor_reloaded.turn_on
    data_template:
      ambient_extract_url: "http://enigma2/grab?format=png&mode=video&r=96"
      entity_id:
        - light.living_room_zha_group_0x0001
      transition: 0.3
      brightness_auto: true
  - delay:
      hours: 0
      minutes: 0
      seconds: 0
      milliseconds: 350
  - service: ambient_extractor_reloaded.turn_on
    data_template:
      ambient_extract_url: "http://enigma2/grab?format=png&mode=video&r=96"
      entity_id:
        - light.living_room_zha_group_0x0001
      transition: 0.3
      brightness_auto: true
mode: single
```

Left, right and ceiling using crop_*.

```yaml
alias: Ambient Light enigma2
description: ""
trigger:
  - platform: time_pattern
    seconds: "*"
    minutes: "*"
    hours: "*"
condition:
  - condition: state
    entity_id: media_player.enigma2
    state: playing
action:
  - service: ambient_extractor_reloaded.turn_on
    data_template:
      ambient_extract_url: "http://enigma2/grab?format=png&mode=video&r=64"
      entity_id:
        - light.living_room_tv_left
      transition: 0.6
      brightness_auto: true
      crop_left: 0
      crop_top: 0
      crop_width: 25
      crop_height: 100
  - delay:
      hours: 0
      minutes: 0
      seconds: 0
      milliseconds: 200
  - service: ambient_extractor_reloaded.turn_on
    data_template:
      ambient_extract_url: "http://enigma2/grab?format=png&mode=video&r=64"
      entity_id:
        - light.living_room_tv_right
      transition: 0.6
      brightness_auto: true
      crop_left: 75
      crop_width: 25
      crop_height: 100
  - delay:
      hours: 0
      minutes: 0
      seconds: 0
      milliseconds: 200
  - service: ambient_extractor_reloaded.turn_on
    data_template:
      ambient_extract_url: "http://enigma2/grab?format=png&mode=video&r=64"
      entity_id:
        - light.living_room_ceiling
      transition: 0.6
      brightness_auto: true
      crop_width: 100
      crop_height: 35
mode: single
```

`crop_width` and `crop_height` need values > 0 or cropping is disabled.

#### Using slower sources
```yaml
alias: Ambient Light FireTV
description: ""
trigger:
  - platform: state
    entity_id: media_player.firetv
action:
  - service: ambient_extractor_reloaded.turn_on
    data_template:
      ambient_extract_url: "http://127.0.0.1:8123{{ states.media_player.firetv.attributes.entity_picture }}"
      entity_id:
        - light.living_room_floor_lamp
      transition: 2
      brightness_auto: true
      color_temperature: 5000
mode: single
```

### Image source examples

See `docs/` for simple examples of screenshot web API servers. Make sure to only use it in your private network (like on your gaming PC).

