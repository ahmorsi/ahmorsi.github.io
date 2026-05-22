---
title: "Integrating Mawaqit Prayer Times into Home Assistant"
date: 2026-05-22 13:13:30 +0200
categories: [Smart Home]
tags: [home-assistant, mawaqit, automation, hacs, iot]
---

[Mawaqit](https://mawaqit.net) is a platform mosques use to manage and broadcast prayer times. Their unofficial Home Assistant integration pulls those times directly from your nearest mosque and exposes them as sensors — giving you accurate, locally-sourced prayer schedules to drive any automation you can think of.

## Prerequisites

- Home Assistant (any recent version) — if you're starting from scratch, see [Installing Home Assistant on a Raspberry Pi](/posts/home-assistant-raspberry-pi/)
- HACS installed — covered in the install post above, or see the [HACS docs](https://hacs.xyz/docs/setup/download)
- A free account at [mawaqit.net](https://mawaqit.net)
- HA configured with your GPS coordinates (Settings → System → General → scroll to "Home location")

The integration uses your coordinates to find mosques within 20 km, so the location config is required.

## Installation via HACS

HACS doesn't list Mawaqit in its default catalogue, so you need to add it as a custom repository.

1. Open HACS → click the three-dot menu (top right) → **Custom repositories**
2. Paste the repo URL: `https://github.com/mawaqit/home-assistant`
3. Set category to **Integration** → click **Add**
4. Search for `Mawaqit` in HACS → click **Download**
5. Restart Home Assistant

## Manual Installation (no HACS)

If you're running HA in a container or prefer direct file access:

```bash
# from your HA config directory
mkdir -p custom_components
git clone https://github.com/mawaqit/home-assistant /tmp/mawaqit
cp -r /tmp/mawaqit/custom_components/mawaqit custom_components/
```

Then restart Home Assistant.

## Configuration

1. Go to **Settings → Devices & Services → Add Integration**
2. Search for `Mawaqit` and select it
3. Enter your mawaqit.net credentials
4. Choose your mosque from the list — it shows options within 20 km of your home coordinates
5. Done — sensors appear immediately, no `configuration.yaml` edits needed

## What You Get: 14 Sensors

| Sensor | Description |
|---|---|
| `sensor.fajr` | Fajr prayer time |
| `sensor.shuruq` | Sunrise |
| `sensor.dhuhr` | Dhuhr prayer time |
| `sensor.asr` | Asr prayer time |
| `sensor.maghrib` | Maghrib prayer time |
| `sensor.isha` | Isha prayer time |
| `sensor.fajr_iqama` … `sensor.isha_iqama` | Iqama times for each prayer |
| `sensor.jumua` | Friday Jumu'a time |
| `sensor.mosque_info` | Full mosque metadata |

All times are strings in `HH:MM` format and update daily.

## Example Automations

**Play the Adhan at prayer time:**

```yaml
automation:
  - alias: "Adhan at Maghrib"
    trigger:
      - platform: template
        value_template: >
          {{ states('sensor.maghrib') == now().strftime('%H:%M') }}
    action:
      - service: media_player.play_media
        target:
          entity_id: media_player.living_room_speaker
        data:
          media_content_id: /local/adhan.mp3
          media_content_type: music
```

**Dim lights 10 minutes before Fajr:**

```yaml
automation:
  - alias: "Prepare for Fajr"
    trigger:
      - platform: template
        value_template: >
          {% set fajr = states('sensor.fajr') %}
          {% set prep = (today_at(fajr) - timedelta(minutes=10)).strftime('%H:%M') %}
          {{ prep == now().strftime('%H:%M') }}
    action:
      - service: light.turn_on
        target:
          entity_id: light.bedroom
        data:
          brightness_pct: 20
          kelvin: 2700
```

**Do Not Disturb during prayer windows:**

```yaml
automation:
  - alias: "Silence notifications at Isha Iqama"
    trigger:
      - platform: template
        value_template: >
          {{ states('sensor.isha_iqama') == now().strftime('%H:%M') }}
    action:
      - service: notify.mobile_app_your_phone
        data:
          message: command_dnd
          data:
            command: turn_on
```

## Tips

- **Time drift**: the sensors give you the mosque's announced times, which may differ slightly from calculated times. This is intentional — it mirrors what the mosque actually broadcasts.
- **Multiple mosques**: you can add the integration more than once if you want sensors from a second mosque (e.g. comparing Jumu'a times).
- **Templating gotcha**: prayer times are strings, not `datetime` objects. Use `today_at(states('sensor.fajr'))` to convert them for offset calculations.
