#  Neoom BEAAM – Home Assistant Integration
Eine vollständige Home Assistant Integration für das Neoom BEAAM Energiemanagementsystem – ohne HACS, ohne externe Abhängigkeiten, nur mit nativen HA-Mitteln.

![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.x-blue?logo=homeassistant)
![YAML](https://img.shields.io/badge/Config-YAML-orange)
![License](https://img.shields.io/badge/License-MIT-green)
![HACS](https://img.shields.io/badge/HACS-not%20required-brightgreen)
[![Buy Me A Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-Donate-yellow?logo=buymeacoffee)](https://www.buymeacoffee.com/vmaddin)

---
## 📋 Inhaltsverzeichnis

- [Features](#features)
- [Voraussetzungen](#voraussetzungen)
- [Dateistruktur](#dateistruktur)
- [Installation](#installation)
- [Dashboard](#dashboard)
- [Sensoren](#sensoren)
- [Einsparungsberechnung](#einsparungsberechnung)
- [Notstromreserve steuern](#notstromreserve-steuern)
- [Tages- und Monatswerte](#tages--und-monatswerte)
- [Bekannte Eigenheiten der API](#bekannte-eigenheiten-der-api)

---

## ✨ Features

- **Echtzeit Energiefluss** – PV-Leistung, Hausverbrauch, Netzbezug/-einspeisung, Batterie
- **DC String Übersicht** – Spannung, Strom, Leistung und Auslastung (% von Wp) für String 1 & 2
- **Einsparungsberechnung** – Eigenverbrauchsersparnis, Einspeiseerlöse, Amortisation
- **Tages-, Monats-, Vormonats- und Gesamtwerte** via `utility_meter`
- **Batterie Details** – SOC, Spannung, Strom, Betriebsmodus, geladene/entladene Energie
- **Netzzähler** – Wirkleistung, Phasen L1/L2/L3, Netzfrequenz
- **Notstromreserve** – Lesen & Schreiben direkt über die BEAAM API per Dashboard-Slider
- **Kein HACS erforderlich** – ausschließlich native HA-Cards (entity, gauge, history-graph)

---

## 🔧 Voraussetzungen

- Home Assistant (getestet ab 2026.x)
- Neoom BEAAM im lokalen Netzwerk erreichbar
- BEAAM API Token (`sk_beaam_...`) – erstellen unter: BEAAM Web-UI → Einstellungen → API
- Folgende `input_number` Helfer müssen bereits in HA vorhanden sein:
  - `input_number.arbeitspreis` (€/kWh) – aktueller Strompreis
  - `input_number.einspeiseverguetung` (€/kWh) – Einspeisevergütung (20-Jahre-Festpreis)
  - `input_number.anlagenkosten` (€) – Installationskosten für Amortisationsberechnung

---

## 📁 Dateistruktur

```
config/
├── packages/
│   ├── neoom.yaml          ← REST-Sensoren + Template-Sensoren + Notstromreserve
│   ├── neoom_costs.yaml    ← Utility Meter + Einsparungs-Sensoren
│   └── strom.yaml          ← Eingefrorene Tagespreise (für korrekte Tagesberechnung)
├── secrets.yaml            ← API Token eintragen
```

---

## 🚀 Installation

**1. Token in secrets.yaml eintragen**

```yaml
neoom_beaam_token: "Bearer sk_beaam_DEIN_TOKEN_HIER"
```

**2. Packages einbinden**

```yaml
# configuration.yaml
homeassistant:
  packages: !include_dir_named packages
```

**3. Dateien in `/config/packages/` ablegen**

- `neoom.yaml`
- `neoom_costs.yaml`
- `strom.yaml`

**4. HA neu starten**

**5. Dashboard einrichten**

1. HA → Dashboards → „+" → Leeres Dashboard
2. Oben rechts ⋮ → „Dashboard bearbeiten" → ⋮ → „Roh-Konfigurations-Editor"
3. Inhalt aus `neoom_dashboard.yaml` einfügen

---

## 🖥️ Dashboard

| View | Inhalt |
|------|--------|
| ⚡ Echtzeit | PV-Leistung, Hausverbrauch, Netz, Batterie, Autarkie, Zählerstand |
| ☀️ DC Strings | String 1 & 2: Spannung, Strom, Leistung, Auslastung (% von Wp) |
| 💰 Einsparungen | Ersparnis Heute/Monat/Vormonat/Gesamt, Amortisation, Energiezähler |
| 🔋 Batterie | Notstromreserve-Slider, SOC, Spannung, Strom, Verlauf |
| 🔌 Netzzähler | Wirkleistung, Phasen L1/L2/L3, Spannungen, Frequenz, Verlauf |

---

## 📡 Sensoren

### Site State (`/api/v1/site/state`)

| Entity | Beschreibung | Einheit |
|--------|-------------|---------|
| `sensor.neoom_pv_leistung` | PV-Gesamtleistung | W |
| `sensor.neoom_hausverbrauch` | Hausverbrauch (berechnet) | W |
| `sensor.neoom_netzbezug` | Netz (+ Bezug / − Einspeisung) | W |
| `sensor.neoom_batterie_leistung` | Batterie (+ Laden / − Entladen) | W |
| `sensor.neoom_batterie_ladestand` | State of Charge | % |
| `sensor.neoom_autarkie` | Autarkie | % |
| `sensor.neoom_energy_erzeugt` | Erzeugte Energie gesamt | Wh |
| `sensor.neoom_energie_eingespeist` | Eingespeiste Energie gesamt | Wh |
| `sensor.neoom_energie_bezogen` | Netzbezug gesamt | Wh |
| `sensor.neoom_batterie_geladen` | Batterie geladen gesamt | Wh |
| `sensor.neoom_batterie_entladen` | Batterie entladen gesamt | Wh |

### DC Strings (`/api/v1/things/{pvThingId}/states`)

| Entity | Beschreibung | Einheit |
|--------|-------------|---------|
| `sensor.neoom_string_1_spannung` | String 1 DC-Spannung | V |
| `sensor.neoom_string_1_strom` | String 1 DC-Strom | A |
| `sensor.neoom_string_1_leistung` | String 1 Leistung (P=U×I) | W |
| `sensor.neoom_string_1_anteil` | String 1 Anteil Gesamtleistung | % |
| `sensor.neoom_string_1_auslastung` | String 1 Auslastung (% von 3.115 Wp) | % |
| `sensor.neoom_string_2_*` | analog String 2 (max 6.675 Wp) | — |

---

## 💰 Einsparungsberechnung

### Architektur

```
REST-Sensor (Wh, total_increasing)
    ↓ utility_meter (daily / monthly / kein cycle)
    ↓ × Preis im Template-Sensor
= EUR (Heute / Monat / Vormonat / Gesamt)
```

### Preislogik

| Zeitraum | Strompreis | Einspeisepreis |
|---|---|---|
| Heute | `sensor.strompreis_heute_eingefroren` | `sensor.einspeisevergutung_heute_eingefroren` |
| Diesen Monat | `input_number.arbeitspreis` | `input_number.einspeiseverguetung` |
| Vormonat | `last_period` der Monats-Meter × aktueller Preis | — |
| Gesamt | `input_number.arbeitspreis` | `input_number.einspeiseverguetung` |

Der eingefrorene Tagespreis (`strom.yaml`) verhindert dass Tarifänderungen vergangene Tageswerte rückwirkend verändern.

### Historische Korrektur (Offset)

Falls die Sensoren nicht von Anfang an liefen, können fehlende Beträge über Offset-Helfer nachgetragen werden:

```yaml
input_number:
  neoom_ersparnis_eigenverbrauch_offset:
    initial: 848.18   # fehlender Betrag in EUR
  neoom_erloese_einspeisung_offset:
    initial: 274.00
```

---

## 🔋 Notstromreserve steuern

### Technische Details

| Parameter | Wert |
|-----------|------|
| API Endpoint | `POST /api/v1/things/{batteryThingId}/commands` |
| Key | `MIN_SOC_SELF_CONSUMPTION_OPT` |
| Formel | `API-Wert = gewünschte Reserve (%) + 8 (Systemreserve)` |
| Bereich | 0 – 92 % (8 % Systemreserve fest eingebaut) |

### Funktionsweise

1. Dashboard-Slider (`input_number.neoom_notstrom_reserve`) auf gewünschten Wert stellen
2. HA-Automation addiert automatisch 8 (Systemreserve)
3. `rest_command` sendet korrekten Wert an `/commands` Endpoint
4. Neoom App zeigt den neuen Wert sofort

> ⚠️ **Wichtig:** Nur `/commands` schreibt tatsächlich ans Gerät – `/states` POST wird akzeptiert aber vom Gerät ignoriert.

> ⚠️ **Wichtig:** `MIN_SOC` ist **nicht** die Notstromreserve – das ist die interne Systemreserve. Der korrekte Key ist `MIN_SOC_SELF_CONSUMPTION_OPT`.

---

## 📅 Tages- und Monatswerte

Alle Energiewerte werden über `utility_meter` direkt aus den REST-Sensoren berechnet – kein Reboot-Problem da HA `unavailable` automatisch ignoriert.

| Sensor | Beschreibung |
|--------|-------------|
| `sensor.neoom_produktion_taeglich` | Produktion heute (Wh) |
| `sensor.neoom_produktion_monatlich` | Produktion diesen Monat (Wh) |
| `sensor.neoom_produktion_vormonat` | Produktion letzter Monat via `last_period` (Wh) |
| `sensor.neoom_ersparnis_eigenverbrauch_heute` | Ersparnis heute (EUR) |
| `sensor.neoom_ersparnis_gesamt_monat` | Ersparnis diesen Monat (EUR) |
| `sensor.neoom_ersparnis_gesamt_vormonat` | Ersparnis letzter Monat (EUR) |
| `sensor.neoom_ersparnis_gesamt` | Ersparnis gesamt seit Inbetriebnahme (EUR) |

---

## 🔑 Bekannte Eigenheiten der API

| Eigenheit | Details |
|-----------|---------|
| **Write Endpoint** | `POST /api/v1/things/{id}/commands` – schreibend (nicht `/states`!) |
| **OPTIONS/PUT/PATCH** | Alle geben 404 zurück |
| **POST /states** | Wird akzeptiert (200 OK) aber vom Gerät still ignoriert |
| **String-Leistung** | Kein `INPUTS_POWER` Key – Leistung muss berechnet werden: P = U × I |
| **Array-Zugriff** | `VOLTAGES` und `CURRENTS` sind `NUMBER_ARRAY` → Zugriff via `arr[0][0]` / `arr[0][1]` |
| **Umlaute in Namen** | HA generiert entity_ids aus dem `name`-Feld – Umlaute führen zu unerwarteten IDs → immer ASCII verwenden |
| **Verkettete Templates** | Template-Sensoren die andere Template-Sensoren referenzieren liefern beim Reboot falsche Werte → alle Berechnungen direkt inline |
| **utility_meter Quelle** | Nur `total_increasing` REST-Sensoren als Quelle verwenden – Template-Sensoren mit täglichem Reset führen zu falschen Monatssummen |
| **Notstromreserve Offset** | App zeigt `MIN_SOC_SELF_CONSUMPTION_OPT - 8` an – die 8% Systemreserve wird intern abgezogen |

---

## 📐 Thing-IDs (anpassen!)

```yaml
PV-Thing:   cf2735a5-028b-4a4d-a017-60ebe066067f
Inverter:   0e07029a-cc69-493d-a6fe-cffcbd1b0561
Batterie:   0eb84721-5482-4ad4-9e57-4c26d6ee2229
Netzzähler: 51071d8d-3663-4799-8652-00452be62b0a
```

Thing-IDs abrufen:
```bash
curl http://192.168.X.X/api/v1/site/configuration \
  -H "Authorization: Bearer sk_beaam_..."
```

---

## 📜 Lizenz

MIT License – frei verwendbar, anpassbar und weiterzugeben.

---
*Entwickelt mit [Claude](https://claude.ai) · Getestet mit Neoom BEAAM + Home Assistant 2026.x*
---
<a href="https://www.buymeacoffee.com/vmaddin" target="_blank">
  <img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" width="200">
</a>
