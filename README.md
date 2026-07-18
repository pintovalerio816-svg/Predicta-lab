# 🎈 Blank app template

A simple Streamlit app template for you to modify!

[![Open in Streamlit](https://static.streamlit.io/badges/streamlit_badge_black_white.svg)](https://blank-app-template.streamlit.app/)

### How to run it on your own machine

Prerequisite: install `uv` if you don't already have it.

```
$ curl -LsSf https://astral.sh/uv/install.sh | sh
```

1. Sync the dependencies

   ```
   $ uv sync
   ```

2. Run the app

   ```
   $ uv run streamlit run streamlit_app.py
   ```
import streamlit as st
import pandas as pd
import numpy as np
import plotly.graph_objects as go
import plotly.express as px
from scipy.signal import correlate
from scipy.stats import zscore
from sklearn.linear_model import LinearRegression
from fpdf import FPDF

# ============================================================
# CONFIGURAZIONE APP COMMERCIALE
# ============================================================

st.set_page_config(
    page_title="PredictaLab Industry",
    layout="wide",
    page_icon="🏭"
)

# Header commerciale
st.markdown("""
<div style="background-color:#0A3D62;padding:12px;border-radius:8px;margin-bottom:20px">
<h2 style="color:white;text-align:center;margin:0">🏭 PredictaLab Industry — Commercial Edition</h2>
<p style="color:#dcdcdc;text-align:center;margin:0">Predictive Maintenance & Industrial Sensor Intelligence</p>
</div>
""", unsafe_allow_html=True)

# ============================================================
# FUNZIONI DI SUPPORTO
# ============================================================

@st.cache_data
def load_data(file):
    ext = file.name.split(".")[-1].lower()
    if ext == "csv":
        return pd.read_csv(file)
    elif ext in ["xlsx", "xls"]:
        return pd.read_excel(file)
    else:
        st.error("Formato non supportato")
        return None

def compute_rul(series):
    y = series.values
    X = np.arange(len(y)).reshape(-1, 1)
    model = LinearRegression().fit(X, y)
    slope = model.coef_[0]
    if slope >= 0:
        return np.inf
    return abs(model.intercept_ / slope)

def anomaly_score(series):
    z = np.abs(zscore(series))
    return np.clip(np.mean(z) * 10, 0, 100)

# ============================================================
# MENU COMMERCIALE
# ============================================================

st.sidebar.title("📌 Menu")
pagina = st.sidebar.selectbox(
    "Seleziona una pagina",
    [
        "Dashboard",
        "Analisi Sensore",
        "Previsioni & RUL",
        "Anomalie",
        "Multi-Sensore",
        "Alert & Soglie",
        "Importa Dati",
        "Report PDF",
        "About / Licenza"
    ]
)

# ============================================================
# PAGINA: DASHBOARD
# ============================================================

def pagina_dashboard(df):
    st.title("🏠 Dashboard — Stato del Macchinario")

    col1, col2, col3, col4 = st.columns(4)
    col1.metric("Media", f"{df['value'].mean():.2f}")
    col2.metric("Deviazione Std", f"{df['value'].std():.2f}")
    col3.metric("Valore Max", f"{df['value'].max():.2f}")
    col4.metric("Anomaly Score", f"{anomaly_score(df['value']):.1f}")

    fig = go.Figure()
    fig.add_trace(go.Scatter(
        x=df["timestamp"],
        y=df["value"],
        mode="lines",
        line=dict(color="#1B9CFC"),
        name="Valore Sensore"
    ))
    fig.update_layout(title="Andamento Sensore", height=400)
    st.plotly_chart(fig, use_container_width=True)# ============================================================
# PAGINA: ANALISI SENSORE (COMMERCIAL EDITION)
# ============================================================

def pagina_analisi(df):
    st.title("📊 Analisi Sensore — Insight Operativi")

    st.markdown("""
    <p style="color:#555;font-size:16px">
    Questa sezione fornisce una panoramica statistica del comportamento del sensore,
    utile per manutentori, ingegneri di processo e responsabili di produzione.
    </p>
    """, unsafe_allow_html=True)

    st.subheader("Statistiche principali")
    col1, col2, col3, col4 = st.columns(4)
    col1.metric("Media", f"{df['value'].mean():.2f}")
    col2.metric("Mediana", f"{df['value'].median():.2f}")
    col3.metric("Varianza# ============================================================
# PAGINA: ANOMALIE (COMMERCIAL EDITION)
# ============================================================

def pagina_anomalie(df):
    st.title("⚠️ Rilevamento Anomalie — Controllo Qualità Avanzato")

    st.markdown("""
    <p style="color:#555;font-size:16px">
    PredictaLab utilizza quattro metodi di rilevamento anomalie per garantire
    un monitoraggio affidabile del macchinario e ridurre i fermi non programm# ============================================================
# PAGINA: ALERT & SOGLIE (COMMERCIAL EDITION)
# ============================================================

def pagina_alert(df):
    st.title("🔔 Alert & Soglie Dinamiche — Monitoraggio in Tempo Reale")

    st.markdown("""
    <p style="color:#555;font-size:16px">
    PredictaLab genera alert automatici basati su soglie dinamiche calcolate
    direttamente dai dati del macchinario, garantendo un monitoraggio affidabile
    e riducendo i falsi positivi.
    </p>
    """, unsafe_allow_html=True)

    media = df["value"].mean()
    sigma = df["value"].std()

    soglia_alta = media + 2 * sigma
    soglia_bassa = media - 2 * sigma

    col1, col2, col3 = st.columns(3)
    col1.metric("Media", f"{media:.2f}")
    col2.metric("Deviazione Std", f"{sigma:.2f}")
    col3.metric("Soglia Alta", f"{soglia_alta:.2f}")

    st.subheader("Alert Automatici")

    df["alert"] = df["value"]
