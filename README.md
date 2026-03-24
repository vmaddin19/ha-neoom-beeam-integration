#  Neoom BEAAM – Home Assistant Integration
Eine vollständige Home Assistant Integration für das **Neoom BEAAM** Energiemanagementsystem – ohne HACS, ohne externe Abhängigkeiten, nur mit nativen HA-Mitteln.

![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.x-blue?logo=homeassistant)
![YAML](https://img.shields.io/badge/Config-YAML-orange)
![License](https://img.shields.io/badge/License-MIT-green)
![HACS](https://img.shields.io/badge/HACS-not%20required-brightgreen)

---
## 📋 Inhaltsverzeichnis

- [Features](#features)
- [Voraussetzungen](#voraussetzungen)
- [Installation](#installation)
- [Dashboard](#dashboard)
- [Sensoren](#sensoren)
- [Notstromreserve steuern](#notstromreserve-steuern)
- [Bekannte Eigenheiten der API](#bekannte-eigenheiten-der-api)
- [Dateiübersicht](#dateiübersicht)

---

---

## ✨ Features

- **Echtzeit Energiefluss** – PV-Leistung, Hausverbrauch, Netzbezug/-einspeisung, Batterie
- **DC String Übersicht** – Spannung, Strom, Leistung und Auslastung (% von Wp) für String 1 & 2
- **Einsparungsberechnung** – Eigenverbrauchsersparnis, Einspeiseerlöse, Amortisation, monatliche Rücklage
- **Batterie Details** – SOC, Spannung, Strom, Betriebs­modus, geladene/entladene Energie
- **Netzzähler** – Wirkleistung, Phasen L1/L2/L3, Netzfrequenz
- **Notstromreserve** – Lesen & Schreiben direkt über die BEAAM API per Dashboard-Slider
- **Kein HACS erforderlich** – ausschließlich native HA-Cards (entity, gauge, history-graph)

---

## 🔧 Voraussetzungen

- Home Assistant (getestet ab 2024.x)
- Neoom BEAAM ist im lokalen Netzwerk erreichbar
- BEAAM API Token (`sk_beaam_...`) – erstellen unter: BEAAM Web-UI → Einstellungen → API
- Folgende `input_number` Helfer müssen bereits in HA vorhanden sein:
  - `input_number.arbeitspreis` (€/kWh)
  - `input_number.einspeiseverguetung` (€/kWh)
  - `input_number.anlagenkosten` (€)

---

## 🚀 Installation

### 1. Token in secrets.yaml eintragen

```yaml
# secrets.yaml
neoom_beaam_token: "Bearer sk_beaam_DEIN_TOKEN_HIER"
```

### 2. Package einbinden

```yaml
# configuration.yaml
homeassistant:
  packages:
    neoom: !include packages/neoom.yaml
```

### 3. Dateien kopieren

```
config/
├── packages/
│   └── neoom.yaml                  ← Haupt-Konfiguration (REST-Sensoren + Templates)
├── secrets.yaml                    ← Token eintragen
```

Inhalte aus `neoom_einsparungs_sensoren.yaml` und `neoom_notstrom_config.yaml` in `packages/neoom.yaml` einfügen (siehe Abschnitt [Dateiübersicht](#dateiübersicht)). Bitte auf die richtige IP und Thing-IDs in den YAML-Dateien achten.

### 4. HA neu starten

```
Einstellungen → System → Neu starten
```

### 5. Dashboard einrichten

1. HA → Dashboards → „+" → Leeres Dashboard erstellen
2. Oben rechts ⋮ → „Dashboard bearbeiten" → ⋮ → „Roh-Konfigurations-Editor"
3. Inhalt aus `neoom_dashboard.yaml` einfügen und speichern

---

## 🖥️ Dashboard
Das Dashboard besteht aus **5 Views**:

| View | Inhalt |
|------|--------|
| ⚡ Echtzeit | PV-Leistung, Hausverbrauch, Netz, Batterie, Autarkie-Gauge, Zählerstand |
| ☀️ DC Strings | String 1 & 2 mit Spannung, Strom, Leistung, Auslastung (% von Wp) |
| 💰 Einsparungen | Ersparnis, Erlöse, Amortisation, monatliche Rücklage, Energiezähler |
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
| `sensor.neoom_energie_erzeugt` | Erzeugte Energie gesamt | Wh |
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

### Einsparungen (Template-Sensoren)

| Entity | Beschreibung | Einheit |
|--------|-------------|---------|
| `sensor.neoom_ersparnis_eigenverbrauch` | Ersparnis durch Eigenverbrauch | EUR |
| `sensor.neoom_erloese_einspeisung` | Erlöse aus Einspeisung | EUR |
| `sensor.neoom_ersparnis_gesamt` | Gesamtersparnis | EUR |
| `sensor.neoom_amortisation_prozent` | Amortisationsfortschritt | % |
| `sensor.neoom_ruecklage_monat` | Empfohlene monatliche Rücklage | EUR |
| `sensor.neoom_notstrom_reserve_kwh` | Notstromreserve in kWh | kWh |
| `sensor.neoom_optimierung_kwh` | Verfügbar für Optimierung | kWh |

---

## 🔋 Notstromreserve steuern

Die Notstromreserve kann direkt über einen Dashboard-Slider gesetzt werden. Bitte die Gesamtkapazität der Batterie anpassen. Die aktuelle Konfiguration ist für 2 Batteriemodule

### Technische Details

| Parameter | Wert |
|-----------|------|
| API Endpoint | `POST /api/v1/things/{batteryThingId}/commands` |
| Key | `MIN_SOC_SELF_CONSUMPTION_OPT` |
| Formel | `API-Wert = gewünschte Reserve (%) + 8 (Systemreserve)` |
| Bereich | 0 – 92 % (8 % Systemreserve fest eingebaut) |

> ⚠️ **Wichtig:** Der `/states` POST-Endpoint wird vom Gerät akzeptiert aber ignoriert.
> Nur `/commands` schreibt den Wert tatsächlich ans Gerät.

> ⚠️ **Wichtig:** `MIN_SOC` ist **nicht** die Notstromreserve – das ist die interne Systemreserve.
> Der korrekte Key ist `MIN_SOC_SELF_CONSUMPTION_OPT`.

### Funktionsweise

1. Dashboard-Slider (`input_number.neoom_notstrom_reserve`) auf gewünschten Wert stellen
2. HA-Automation erkennt Änderung und addiert automatisch 8 (Systemreserve)
3. `rest_command` sendet korrekten Wert an `/commands` Endpoint
4. Neoom App und BEAAM zeigen den neuen Wert sofort

---

## 🔑 Bekannte Eigenheiten der API

| Eigenheit | Details |
|-----------|---------|
| **Read-only Endpoint** | `GET /api/v1/things/{id}/states` – nur lesend |
| **Write Endpoint** | `POST /api/v1/things/{id}/commands` – schreibend (nicht `/states`!) |
| **OPTIONS/PUT/PATCH** | Alle geben 404 zurück |
| **String-Leistung** | Kein `INPUTS_POWER` Key – Leistung muss berechnet werden: P = U × I |
| **Array-Zugriff** | `VOLTAGES` und `CURRENTS` sind `NUMBER_ARRAY` → Zugriff via `arr[0][0]` / `arr[0][1]` |
| **Umlaute in Namen** | HA generiert entity_ids aus dem `name`-Feld – Umlaute führen zu unerwarteten IDs (z.B. `erl_se` statt `erloese`) → immer ASCII verwenden |
| **Verkettete Templates** | Template-Sensoren die andere Template-Sensoren referenzieren liefern falsche Werte → alle Berechnungen direkt inline |
| **Statistic Cards** | Benötigen `state_class: total_increasing` und sammeln erst ab Einrichtung Daten → `entity`-Cards als sofortige Alternative |

---

## 📁 Dateiübersicht

| Datei | Inhalt | Einbinden |
|-------|--------|-----------|
| `neoom_beaam_configuration.yaml` | REST-Sensoren (5 Blöcke) + Template-Sensoren | Als Package in `configuration.yaml` |
| `neoom_einsparungs_sensoren.yaml` | Einsparungs- & Auslastungs-Sensoren | Unter `template: - sensor:` in `neoom.yaml` anhängen |
| `neoom_notstrom_config.yaml` | `input_number`, `rest_command`, Automation | Auf oberster Ebene in `neoom.yaml` einfügen |
| `neoom_dashboard.yaml` | Lovelace Dashboard (5 Views) | Im Roh-Konfigurations-Editor einfügen |

---

## 📐 Thing-IDs (Beispiel – anpassen!)

```yaml
PV-Thing:       cf2735a5-028b-4a4d-a017-60ebe066067f
Inverter:       0e07029a-cc69-493d-a6fe-cffcbd1b0561
Batterie:       0eb84721-5482-4ad4-9e57-4c26d6ee2229
Netzzähler:     51071d8d-3663-4799-8652-00452be62b0a
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
