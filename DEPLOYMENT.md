# Deployment-Anleitung für Render

## Übersicht

Diese Anleitung beschreibt, wie eine Anwendung mit separatem Backend (FastAPI) und Frontend (Streamlit) auf Render.com bereitgestellt wird.

---

## Architektur

- **Backend (FastAPI)**: Bereitgestellt als Web Service bei Render.
- **Frontend (Streamlit)**: Bereitgestellt als separater Web Service bei Render, kommuniziert mit dem Backend über HTTP.

---

## Vorbereitung

Zwei Dienste auf Render müssen eingerichtet werden:

### 1. Backend-Dienst (FastAPI)

- **Build Command**:
  ```bash
  pip install -r requirements.txt
  ```
- **Start Command**:
  ```bash
  uvicorn app.main:app --host 0.0.0.0 --port $PORT
  ```

### 2. Frontend-Dienst (Streamlit)

- **Build Command**:
  ```bash
  pip install -r requirements.txt
  ```
- **Start Command**:
  ```bash
  streamlit run streamlit_app.py --server.port=$PORT --server.address=0.0.0.0
  ```

---

## API-Verbindung konfigurieren

Die Streamlit-App darf sich im Deployment nicht mit `localhost:8000` verbinden. Stattdessen ist die öffentliche URL des Backend-Dienstes anzugeben.

### Zwischenlösung: Secrets-Datei verwenden

Im vorliegenden Projekt wurde die API-URL in der Datei `.streamlit/secrets.toml` hinterlegt:

```toml
STREAMLIT_API_URL = "https://vdss-4ovd.onrender.com"
```

**Hinweis:** Diese Methode erfordert ein erneutes Deployment mit der Änderungen der URL. Für produktive Umgebungen wird empfohlen, stattdessen Umgebungsvariablen zu nutzen.

### Alternative: Environment Variable setzen

In den Environment Settings des Streamlit-Dienstes bei Render kann alternativ folgende Variable definiert werden:

```env
STREAMLIT_API_URL=https://<BACKEND-DIENSTNAME>.onrender.com
```

Beispiel:

```env
STREAMLIT_API_URL=https://vdss-4ovd.onrender.com
```

Die Anwendung prüft sowohl `st.secrets['STREAMLIT_API_URL']` als auch `os.getenv('STREAMLIT_API_URL')`, wodurch beide Varianten unterstützt werden.

### Debug-Modus aktivieren (optional)

Zum Testen kann folgende Variable gesetzt werden:

```env
DEBUG=true
```

Im Debug-Modus werden alle erkannten Konfigurationsquellen in der Streamlit-Oberfläche ausgegeben:

- `Secrets URL`
- `Env Variable`
- `Cloud Mode`

---

## Konfigurationsreihenfolge

Die Anwendung ermittelt die API-URL in folgender Reihenfolge:

1. Streamlit Secrets (`.streamlit/secrets.toml`)
2. Environment Variable (`STREAMLIT_API_URL`)
3. Cloud Auto-Detection (Render)
4. Fallback auf localhost (`http://localhost:8000`)

---

## Troubleshooting

### Fehler: "Connection refused"

- Überprüfen, ob das Backend erreichbar ist
- Sicherstellen, dass die korrekte URL verwendet wird
- `DEBUG=true` aktivieren, um die verwendete URL anzuzeigen

### Backend ist nicht erreichbar

- Die Anwendung schaltet automatisch in den Mock-Modus
- Um dies zu unterbinden: `MOCK_MODE=false` setzen

---

## Hinweise

- Die Angabe von `--server.address=0.0.0.0` ist zwingend erforderlich, damit Render externe Zugriffe zulässt.
- Beide Render-Dienste müssen öffentlich erreichbar sein.
- Die Port-Variable `$PORT` wird automatisch von Render bereitgestellt.

