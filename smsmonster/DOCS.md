# SMS Monster add-on

Turns one or more SIM800C USB modems plugged into your Home Assistant host into an SMS gateway with a web UI (via Ingress), a REST API and webhooks.

## Hardware

- SIM800C breakout with its own USB-serial chip (CH340 / CP2102 / PL2303), one USB port per modem.
- A SIM card with PIN disabled, or set the `sim_pin` option.
- A stable 5 V supply (SIM800 draws up to 2 A bursts; use a powered hub on a Raspberry Pi).

## Options

| Option | Default | Description |
|---|---|---|
| `devices` | `auto` | `auto` scans `/dev/serial/by-id`, `/dev/ttyUSB*`, `/dev/ttyACM*`. Or a comma separated list of device paths. |
| `baudrates` | `115200,9600` | Baud rates tried when probing a modem. |
| `sim_pin` | | PIN entered automatically if the SIM asks for one. |
| `api_token` | | When set, `/api/*` requires `Authorization: Bearer <token>` (Ingress is exempt). **Set this if you expose port 8080**; anyone with the token or open port can read all SMS and fully control the modem. |
| `log_level` | `info` | Log verbosity. |
| `hangup_calls` | `true` | Reject incoming calls after logging them. |
| `delivery_reports` | `false` | Request SMS delivery reports (status becomes `delivered`). |
| `ha_integration` | `true` | Create sensors in Home Assistant for modems and messages and fire `smsmonster_*` events (see below). |

## Entities and events

The add-on talks to the Home Assistant API itself, nothing to configure. It keeps these sensors up to date
(re-pushed every minute, so they come back after a Home Assistant restart):

| Entity | State | Attributes |
|---|---|---|
| `sensor.sms_monster_modem_<imei>` | `ready`, `connecting`, `disconnected`, `sim_pin_required`, … | operator, registration, signal, IMEI/ICCID/IMSI, model, firmware, name, phone_number, last_error, counters |
| `sensor.sms_monster_modem_<imei>_signal` | signal strength in % | rssi, signal_dbm |
| `sensor.sms_monster_modems` | number of modems | `ready` count, list of modems |
| `sensor.sms_monster_received` / `_sent` / `_failed` / `_queued` | message counters | |
| `sensor.sms_monster_last_received` / `_last_sent` / `_last_failed` / `_last_call` | timestamp of the latest one | id, number, text, modem_id, status, error |

Every message also fires an event carrying the full message as `trigger.event.data`: `smsmonster_message_received`,
`smsmonster_message_sent`, `smsmonster_message_failed`, `smsmonster_call_received`, and `smsmonster_modem_removed`.

```yaml
automation:
  - alias: Forward SMS to phone
    trigger:
      - platform: event
        event_type: smsmonster_message_received
    action:
      - service: notify.mobile_app_phone
        data:
          title: "SMS from {{ trigger.event.data.number }}"
          message: "{{ trigger.event.data.text }}"
```

## Using it from Home Assistant

Send an SMS with a `rest_command`:

```yaml
rest_command:
  send_sms:
    url: "http://<host>:8080/api/messages"
    method: POST
    headers:
      Authorization: "Bearer YOUR_TOKEN"
    content_type: application/json
    payload: '{"to": "{{ to }}", "text": "{{ text }}"}'
```

Receiving SMS needs no webhook: use the `smsmonster_message_received` event above. A webhook still works if you prefer it:
add one in the add-on UI pointing at a Home Assistant webhook trigger
(`http://homeassistant.local:8123/api/webhook/<your-id>`) for the `message.received` event.
The payload is `{"event": "message.received", "data": {"number": "...", "text": "...", ...}}`,
so an automation can use `{{ trigger.json.data.number }}` and `{{ trigger.json.data.text }}`.
