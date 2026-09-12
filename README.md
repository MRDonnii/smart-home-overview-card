# Smart Home Overview Card

Én stor, opinioneret forside til Home Assistant: klokke/vejr-hero, en roterende alarm-ticker, fire nøgletal-fliser (husforbrug, elbil, elpris, pool), fire systemfliser (kæledyr, sikkerhed, varme, indstillinger), et live kamera-grid, en familie/tilstedeværelses-grid og en elpris-graf med dag/i morgen/uge-faner — alt sammen i ét kort.

Layoutet er fast (det er kortets design), men **alt indhold er konfigurerbart**: hvilke entiteter der bruges, hvilke alarmer der vises, hvilke personer der indgår, hvilke kameraer der roterer igennem, og hvor hver flise navigerer hen. Har en fuld visuel editor (felter + JSON), så det kan sættes op uden at skrive YAML i hånden.

```yaml
type: custom:smart-home-overview-card
title: Hjemmet
```

Sådan alene giver kortet dig en fungerende demo-opsætning med eksempel-entiteter — brug editoren (eller YAML) til at pege felterne på dine egne.

## Konfiguration

### Simple felter

| Felt | Type | Standard |
|---|---|---|
| `title` | tekst | "Hjemmet" — vises i toppen af hero'en |
| `pet_name` | tekst | "Kæledyr" — titel på kæledyr-flisen |
| `car_name` | tekst | "Bilen" — titel på elbil-flisen |
| `currency` | tekst | "kr" — bruges i pris-visninger |

### `entities` (objekt, alle nøgler valgfrie — overskriv kun det du bruger)

| Nøgle | Standard | Bruges til |
|---|---|---|
| `houseMode` | `input_select.house_mode` | Hero-mode (Hjemme/Gæster/Ferie/Nat/Stille) |
| `guestOverride` / `holidayOverride` | `input_boolean.house_mode_guest_override` / `_holiday_override` | Override af hero-mode |
| `alarm` / `secondaryAlarm` | `alarm_control_panel.home_alarm` / `secondary_alarm` | Alarmpanel-status |
| `weather` | `weather.home` | Vejr-hero |
| `powerTotal` | `sensor.house_power` | Samlet effekt (W) til husforbrug-flisen |
| `evPowerKw` | `sensor.ev_charger_power_kw` | Elbil-ladeeffekt, trækkes fra `powerTotal` for at vise "hus uden bil" |
| `homeEnergyToday` | `sensor.home_energy_today` | kWh i dag, hus uden bil |
| `evBattery` / `evEnergyToday` / `evChargeState` | `sensor.ev_battery` / `sensor.ev_energy_today` / `sensor.ev_charger_state` | Elbil-flisen |
| `priceNow` / `priceTomorrowAvailable` / `priceForecast` / `priceForecastDaySelector` | `sensor.electricity_price_now` m.fl. | Elpris-flise og -graf (forventer et `attributes.prices`-array a la Stromligning/EDS) |
| `poolTemp` / `poolOccupied` / `poolPumpRunning` | `sensor.pool_temperature` m.fl. | Pool-flisen |
| `petMode` / `petMorning` / `petMidday` / `petEvening` | `select.pet_feeder_mode` m.fl. | Kæledyrs-flisen |
| `lockFront` / `lockUtility` / `lockGarage` / `terraceDoorContact` / `gateContact` | `lock.front_door` m.fl. | Sikkerheds-flisen (kun de nøgler du udfylder tælles med) |
| `openWindows` | `sensor.open_windows` | Antal åbne vinduer i sikkerheds-flisen |
| `hotWaterStatus` / `heatingState` / `heatingPressure` / `heatingValveHeating` / `heatingValveIdle` / `heatingEnergyToday` / `cheapestHeatingStatus` | `sensor.hot_water_status` m.fl. | Varme-flisen |
| `automationsOffCount` / `kioskEditMode` | `sensor.automations_off_count` / `input_boolean.kiosk_edit_mode` | Indstillings-flisen |
| `applianceWasher` / `applianceDryer` / `applianceDishwasher` | `binary_sensor.washer_running` m.fl. | Aktive hvidevarer-tæller |
| `nightLockWindow` | `binary_sensor.night_lock_warning_window` | Se `night_locks` nedenfor |

### `paths` (objekt, navigationsmål for hver flise/knap)

`weather`, `energy`, `ev`, `electricityPrice`, `pool`, `pet`, `security`, `heating`, `settings`, `cameras` — standard er simple ruter som `/energy`, `/security` osv.

### `price_thresholds`

`{ medium, high, critical }` i kr/kWh (standard `2.05` / `3.99` / `5.49`) — styrer farve og label på elpris-flisen og -graf-barerne.

### `alerts` (array)

Hver alarm er `{ entity, kind, title, detail, icon, action, toggle }`:
- `kind: "on"` — aktiv når entity er `on`.
- `kind: "battery"` — aktiv når `state < action` (så `action` er tærsklen, et tal).
- `kind: "not-clear"` — aktiv når state ikke er en af `none/no_error/unknown/unavailable/ok/idle`.
- `action` — enten et tal (batteri-tærskel) eller en navigations-sti (tekst) som ticker-kortet navigerer til ved klik.
- `toggle` — sæt `true` hvis alarmen selv skal kunne slås fra ved langt tryk (kalder `input_boolean.toggle` på `entity`).

Derudover scanner kortet automatisk alle `binary_sensor.*_smoke_alarm_detected`, `*_co_alarm_detected` og `*_(glass_break|siren)_detected` i hele installationen — de behøver ikke stå i `alerts`.

### `night_locks` / `night_extra_contacts` / `nightLockWindow`

Når `entities.nightLockWindow` er `on` (dit eget "det er sent og dørene bør være låst"-vindue, fx bygget med en `time`-trigger-automation), tjekker kortet:
- `night_locks` — array af lock-entity-id'er; alarmerer på enhver der ikke er `locked`.
- `night_extra_contacts` — array af `{ entity, label, open_state, icon }` for andre kontakter (skur, garageport) hvor `open_state` (standard `"on"`) er den tilstand der tæller som "åben".

### `heating_fault_entities` (array af entity-id'er)

Alle `binary_sensor`'er der, når de er `on`, sætter varme-flisen i fejl-tilstand.

### `people` (array)

Hver person er `{ name, entity, city, battery, trip, path, hold, reminder }`:
- `entity` — en `person.*`-entity (bruges til billede og hjemme/ude-status).
- `city` — valgfri entity med et menneskelæsbart sted (falder tilbage til "Ude"/state).
- `battery` — valgfri batteri-sensor for personens enhed.
- `trip` — valgfri sensor med hjemkørselstid i minutter.
- `path` — navigation ved klik på personen.
- `hold` — valgfri `button.*`-entity der trykkes ved langt tryk (fx "opdater lokation nu").
- `reminder` — valgfri binary_sensor; viser et lille "!"-badge på personen når den er `on`.

### `camera_groups` (array)

Hver gruppe er `{ label, selector_entity, path, cameras }`. `selector_entity` er en sensor hvis state matcher ét af kameraernes `key` — det bestemmer hvilket kamera i gruppen der vises lige nu (fx en rotation styret af et automation-script). Hver post i `cameras` er:

```json
{ "key": "front_door", "name": "Front door", "camera_entity": "camera.front_door", "path": "/cameras/front-door", "detection_prefix": "front_door", "motion_entity": "binary_sensor.front_door_motion", "extra_event_entities": [] }
```

- `detection_prefix` (valgfri, standard = `key`) — kortet udleder automatisk detektions-status fra `binary_sensor.<prefix>_person_detected`, `_animal_detected`, `_vehicle_detected`, `_object_detected`, `_audio_object_detected`, `_doorbell`, `_license_plate_detected`, `_motion` — matcher det navngivningsmønster UniFi Protect-integrationen bruger.
- `motion_entity` (valgfri) — overskriv hvilken sensor der tæller som "bevægelse", hvis den afviger fra `<prefix>_motion`.
- `extra_event_entities` (valgfri array) — ekstra binary_sensors der også tæller som en "hændelse" (fx en lyd-sensor).

Kameraerne mountes live via Home Assistants indbyggede `picture-glance`-kort (kræver `window.loadCardHelpers`, som er tilgængelig i alle moderne HA-frontends).

## Installation

1. Kopiér `smart-home-overview-card.js` til `/config/www/smart-home-overview-card/`.
2. Tilføj som Lovelace-resource: `/local/smart-home-overview-card/smart-home-overview-card.js?v=1`, type `module`.
3. Tilføj kortet via UI'ets kortvælger (den har en `getStubConfig`, så den dukker op med demo-data) eller i YAML, og udfyld dine egne `entities`/`paths`/`alerts`/`people`/`camera_groups`.

## Baggrund

Layoutet (klokke/vejr-hero → alarm-ticker → 4+4 fliser → kamera-grid → familie-grid → prisgraf) er en fast, gennemtestet forside-struktur bygget til et dashboard med mange bygningsautomatiserings-integrationer. Hvis du kun bruger nogle af integrationerne (fx ingen pool, ingen elbil), lader du bare de tilhørende `entities`-nøgler stå på deres defaults — fliserne viser blot "–" for manglende data i stedet for at fejle.
