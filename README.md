# Baustellenverkehrs-Management-System (VDSS)

Ein umfassendes System zur Verwaltung und Visualisierung von Verkehr rund um Baustellen mit interaktiven Karten, Datenanalyse und Simulation.

## Überblick

Das Baustellenverkehrs-Management-System (VDSS - Verkehrsdaten-Simulations-System) ist eine deutschsprachige Webanwendung, die es ermöglicht, Verkehrsauswirkungen von Baustellen zu analysieren, zu simulieren und zu visualisieren. Das System kombiniert reale Verkehrszählungsdaten mit OpenStreetMap-basierten Simulationen für eine umfassende Verkehrsanalyse.

## Hauptfunktionen

### Dashboard
- **Interaktive Verkehrskarte** mit PyDeck-basierter Visualisierung
- **Echtzeit-Verkehrssimulation** basierend auf OSM-Daten und Verkehrszählstellen
- **KPI-Übersicht** mit Tooltips: Gesamtlieferungen, Verkehrsanteile, Verkehrsbelastung
- **Zeitliche Analyse** mit wöchentlichen und stündlichen Verkehrsmustern
- **Animierte Verkehrsverläufe** mit Play/Pause-Funktionalität

### Projekteinrichtung
- **Excel-Upload** für Bauzeitpläne (Material_Lieferungen.csv)
- **GeoJSON-Konfiguration** für Baustellenpolygone und Zufahrtsrouten
- **Verkehrszählstellen-Management** mit CSV-Import
- **Liefertage und -zeiten** Konfiguration
- **Automatische Koordinatenverarbeitung** und Geocoding

### Admin-Panel
- **Projektbearbeitung** und Datenverwaltung
- **Excel-Aktualisierung** mit Validierung
- **Simulationseinstellungen** und -ausführung
- **Datenexport** und Berichtserstellung

### Anwohner-Information
- **Vereinfachte Verkehrsübersicht** für betroffene Anwohner
- **Wochenübersicht** mit Verkehrsstatus (Niedrig/Mäßig/Stark)
- **Bauzeitplan-Integration** mit Material- und Personalprognosen

## 🛠️ Technologie-Stack

### Backend
- **FastAPI** (Python) - Moderne, schnelle Web-API
- **Uvicorn** - ASGI-Server für Produktions-Performance
- **Pydantic** - Datenvalidierung und Serialisierung

### Frontend
- **Streamlit** - Interaktive Web-Benutzeroberfläche
- **PyDeck** - 3D-Kartenvisualisierung mit WebGL
- **Plotly** - Interaktive Diagramme und Grafiken

### Geodaten & Karten
- **OpenStreetMap (OSM)** - Straßennetzwerk-Daten
- **OSMnx** - OSM-Daten-Processing und Netzwerk-Analyse
- **GeoPandas** - Geospatiale Datenverarbeitung
- **PyProj** - Koordinatensystem-Transformationen

### Datenverarbeitung
- **Pandas** - Datenmanipulation und -analyse
- **NumPy** - Numerische Berechnungen
- **OpenPyXL** - Excel-Datei-Verarbeitung

### PDF & Berichte
- **ReportLab** - PDF-Generierung für Berichte

## Projektstruktur

```
VDSS/
├── app/                      # FastAPI Backend
│   ├── main.py              # FastAPI Hauptanwendung
│   ├── api/routers/         # API-Endpunkte
│   │   ├── projects.py      # Projekt-Management
│   │   ├── simulation.py    # Verkehrssimulation
│   │   └── export.py        # Datenexport
│   ├── models/              # Pydantic Datenmodelle
│   │   └── simulation.py    # Simulations-Datenstrukturen
│   └── services/            # Business Logic
│       └── simulation_service.py  # Verkehrssimulation
├── modules/                   # Streamlit Seiten-Module
│   ├── dashboard.py         # Haupt-Dashboard (1372 Zeilen)
│   ├── project_setup.py     # Projekteinrichtung (567 Zeilen)
│   ├── admin.py            # Admin-Panel (386 Zeilen)
│   └── resident_info.py     # Anwohner-Info (382 Zeilen)
├── utils/                   # Hilfsfunktionen
│   ├── map_utils.py         # Karten-Utilities
│   ├── legend_widget.py     # Karten-Legende
│   ├── custom_styles.py     # CSS-Styling
│   └── dashoboard_utils.py  # Dashboard-Hilfsfunktionen
├── data/                    # Datenverzeichnis
│   ├── projects/            # Projektdateien
│   ├── prepared/            # Verarbeitete Daten
│   │   ├── profiles/        # Verkehrszählprofile
│   │   └── osm_cache/       # OSM-Daten-Cache
│   ├── simulations/         # Simulationsergebnisse
│   └── reports/             # Generierte Berichte
├── streamlit_app.py         # Streamlit Hauptanwendung
├── run.py                   # Anwendungs-Starter
└── requirements.txt         # Python-Abhängigkeiten
```

## Installation und Ausführung

### Voraussetzungen
- Python 3.8+
- Git

### 1. Repository klonen
```bash
git clone <repository-url>
cd VDSS
```

### 2. Virtual Environment erstellen
```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate
```

### 3. Abhängigkeiten installieren
```bash
pip install -r requirements.txt
```

### 4. Anwendung starten
```bash
# Beide Services starten (empfohlen):
python run.py

# Oder separat:
python run.py backend   # Nur FastAPI (Port 8000)
python run.py frontend  # Nur Streamlit (Port 8501)
```

Die Anwendung öffnet sich automatisch unter `http://localhost:8501`

## Verkehrssimulations-Logik

### 1. Datengrundlage

**OpenStreetMap-Integration:**
- Automatischer Download von Straßennetzwerk-Daten innerhalb der Projektgrenzen
- Lokales Caching (GeoPackage) zur Performance-Optimierung
- Kapazitätszuordnung basierend auf Straßentyp (`highway`-Tag)

**Verkehrszählstellen:**
- Integration realer Verkehrszählungsdaten aus CSV-Dateien
- Profile pro Zählstelle mit wöchentlichen und stündlichen Mustern
- Primäre und sekundäre Zählstellen-Gewichtung

### 2. Verkehrsvolumen-Simulation

**Mehrstufiger Ansatz:**

1. **Globaler Stundenfaktor (`time_factor_current_hour`)**:
   - Berücksichtigt typische Tagesverläufe (Rush-Hour: 7-9, 16-18 Uhr)
   - Integration realer Überlastung von Verkehrszählstellen
   - Basis: 0.05-0.95 je nach Tageszeit und realer Verkehrslage

2. **Straßentyp-spezifische Nutzung**:
   ```python
   utilization_factors = {
       'motorway': (0.30, 0.85),     # Autobahn: 30-85% Kapazität
       'primary': (0.30, 0.85),      # Hauptstraße: 30-85%
       'residential': (0.03, 0.25),  # Wohnstraße: 3-25%
       'service': (0.02, 0.20)       # Erschließung: 2-20%
   }
   ```

3. **Segment-spezifische Variabilität**:
   - Hash-basierte, stabile Zufallsfaktoren pro Straßensegment
   - Gewährleistet realistische Variation zwischen Straßen gleichen Typs
   - Faktor: 0.3-1.0 basierend auf Segment-ID

4. **Baustellenverkehr-Integration**:
   - Reale Lieferungen aus Bauzeitplan (Excel-Import)
   - Verteilung auf Zufahrtsrouten basierend auf GeoJSON-Geometrie
   - Formel: `1 + ceil(Material_kg / 10)` Lieferungen pro Tag

### 3. Stundliche Verkehrsverteilung

**Lieferverkehr-Muster:**
```python
stundliche_gewichtung = {
    7: 1,   8: 2,   9: 5,
    10: 5,  # Erster Peak
    11: 3,  12: 0,  # Mittagspause
    13: 0,  14: 5,  # Zweiter Peak
    15: 5,  16: 2,  17: 1
}
```

**Zuordnungsverfahren:**
- Multinomiale Zufallsverteilung für ganzzahlige Lieferungen
- Seed basiert auf Datum + Projekt-ID für Reproduzierbarkeit
- Cache-System für konsistente Tageswerte

### 4. Überlastungsberechnung

```python
congestion_level = min(1.0, simulated_volume / segment_capacity)
```

**Visualisierung:**
- Grün (0.0-0.3): Geringer Verkehr
- Gelb (0.3-0.7): Mäßiger Verkehr  
- Rot (0.7-1.0): Starker Verkehr/Stau

## Datenformate

### Excel-Bauzeitplan (Material_Lieferungen.csv)
```csv
Anfangstermin,Material,Personen,Phase,Beschreibung
2024-01-15 08:00,150,5,Fundament,Betonlieferung
2024-01-16 10:00,200,8,Rohbau,Stahlträger
```

### Verkehrszählstellen (counters.csv)
```csv
profile_id,lat,lon,name,display_name
12345_IN,47.3769,8.5417,Bahnhofstrasse,Bahnhofstrasse Richtung Innenstadt
```

### Verkehrsprofile (profiles/{station_id}_{direction}.csv)
```csv
weekday,month,hour,vehicles
Monday,1,8,245
Monday,1,9,378
```

## Konfiguration

### Projekt-Setup
1. **Excel-Upload**: Bauzeitplan mit Material, Personal, Terminen
2. **GeoJSON-Definition**: Baustellenpolygon und Zufahrtsrouten
3. **Verkehrszählstellen**: Auswahl relevanter Messpunkte
4. **Zeitfenster**: Liefertage (Mo-Fr) und -zeiten (6-18 Uhr)

### Performance-Optimierung
- **OSM-Caching**: Lokale GeoPackage-Dateien
- **Wochen-Vorladeung**: Batch-Berechnung für ganze Wochen
- **Session-Cache**: Koordinaten und Profile in `st.session_state`

## 🔧 API-Endpunkte

### Projekte
```
GET  /api/projects/           # Alle Projekte abrufen
POST /api/projects/           # Neues Projekt erstellen
GET  /api/projects/{id}       # Projekt details
PUT  /api/projects/{id}       # Projekt aktualisieren
```

### Simulation
```
POST /api/simulation/run      # Simulation ausführen
GET  /api/simulation/results  # Ergebnisse abrufen
```

### Export
```
POST /api/export/pdf          # PDF-Bericht generieren
```

## Benutzeroberfläche

### Design-Prinzipien
- **Deutsche Lokalisierung**: Vollständig deutschsprachige Oberfläche
- **Responsive Layout**: Anpassbare Widget-Breite (30% Dashboard, 50% Setup)
- **Interaktive Karten**: PyDeck mit 3D-Visualisierung
- **Tooltips**: Kontextuelle Hilfe für KPIs und Karten-Elemente

### Styling
- **Corporate Colors**: Blau (#0F05A0) als Primärfarbe
- **KPI-Cards**: Einheitliches Design mit Hover-Effekten
- **Karten-Legende**: Überlagerung mit Bereichen und Verkehrsstatus
- **Diagramme**: Plotly mit deutscher Lokalisierung

## Debug & Monitoring

### Debug-Modi
```python
DEBUG_COORDS = False  # Koordinaten-Verarbeitung
DEBUG_OSM = False     # OSM-Daten-Abruf
```

### Cache-Management
- **OSM-Cache**: `data/prepared/osm_cache/osm_segments_{hash}.gpkg`
- **Profil-Cache**: Session-basiert für Verkehrszählstellen
- **Wochen-Cache**: `traffic_data_week_{year}_{week}_{project_id}`

## Erweiterbarkeit

### Modulare Architektur
- **Seiten-Module**: Unabhängige Streamlit-Pages
- **Utility-Module**: Wiederverwendbare Funktionen
- **API-Router**: RESTful Backend-Services

### Integrationsmöglichkeiten
- **SUMO**: Für detaillierte Mikroskopische Simulation
- **Real-time APIs**: Live-Verkehrsdaten-Integration
- **GIS-Systeme**: Erweiterte Geodaten-Quellen

## Beitrag zur Entwicklung
- **Manuel Weilenmann** : Entwicklung und Design
- **Gwendoline Vocat** : Entwicklung und Design

### Code-Standards
- **Deutsch**: Kommentare und Dokumentation
- **Type Hints**: Python-Typisierung wo möglich
- **Docstrings**: Funktionsdokumentation
- **Error Handling**: Umfassende Fehlerbehandlung

### Testing
```bash
pytest                        # Unit Tests
pytest-asyncio              # Async API Tests
```

---

**Entwickelt für effizientes Baustellenverkehrs-Management mit modernen Web-Technologien.**


