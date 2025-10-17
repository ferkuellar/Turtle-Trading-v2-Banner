![Turtle Trading v2 Banner](./turtlebanner.png)

# 🐢 Turtle Trading v2 — Rebalanced Long & Short [FK]

**Autor:** [Fernando Cuéllar](https://kuellarfer.com)  
**Versión:** v2.0 (Pine Script® v6)  
**Licencia:** [Mozilla Public License 2.0](https://mozilla.org/MPL/2.0/)  
**Repositorio:** [github.com/fercuellar/TurtleTrading-v2](#)

---

## 📘 Descripción General

**Turtle Trading v2 [FK]** es una reinterpretación institucional y cuantitativa del clásico sistema de **Richard Dennis y William Eckhardt**, actualizada para mercados modernos de alta volatilidad como **criptomonedas, oro y metales**.  
Integra gestión de riesgo profesional, piramidación dinámica, HUD informativo y panel de validación visual ("Semáforo").

---

## ⚙️ Características Principales

- 📈 **Estrategia de ruptura Donchian (20/55 días)**
- 🧠 **Gestión de riesgo dinámica basada en ATR**
- 🟩 **Filtro de tendencia con MA200**
- 🔁 **Piramidación inteligente hasta 3 unidades**
- 🚨 **Alertas automáticas compatibles con webhooks**
- 🖥️ **HUD lateral + semáforo central**
- 🧩 **Código limpio, optimizado para backtesting y bots**

---

## 🧠 Filosofía del Sistema

> “No se trata de adivinar el mercado, sino de gestionar el riesgo mientras sigues la tendencia.”  
> — *Richard Dennis (Turtle Trader)*

El sistema no busca predecir.  
Sigue el flujo del mercado usando **canales Donchian**, gestión de riesgo antifrágil y reglas cuantitativas, dejando correr beneficios y limitando pérdidas.

---

## ⚙️ Parámetros de Configuración

| Grupo | Parámetro | Descripción | Valor por Defecto |
|--------|------------|-------------|------------------|
| **Modo / Reglas** | 20-día solo tras pérdida | Usa canal 20d tras pérdida. | ✅ Activado |
| | Dirección habilitada | “Both”, “Long only”, “Short only”. | Both |
| **Sizing / Riesgo** | Riesgo % por trade | % del capital arriesgado. | 0.5 |
| | Cuenta (USD) | Monto base para sizing. | 3000 |
| **Filtros** | MA200 | Filtro de tendencia activo. | ✅ |
| | Backtest desde | Fecha inicial. | 2020-01-01 |
| **Avanzado** | ATR (N) | Periodos ATR. | 25 |
| | Stop inicial (xN) | Stop = múltiplo de N. | 2.5 |
| | Paso entre adds (xN) | Distancia entre piramidaciones. | 0.75 |
| | Máx. unidades | Acumulación máxima. | 3 |
| **Símbolo** | Paso de cantidad | Incremento mínimo (lot step). | 0.001 |
| | Cantidad mínima | Tamaño mínimo permitido. | 0.001 |
| **Alertas** | Activar alertas | Envía alertas runtime. | ✅ |

---

## 📊 Lógica de Operación

### 1. Entradas
- **LONG:** Ruptura del canal superior Donchian (55d o 20d).  
- **SHORT:** Ruptura del canal inferior.  
- Tras pérdida, usa canal corto (20d).

### 2. Salidas
- Stop inicial = `StopMultN × N`.  
- Stop dinámico = canal Donchian opuesto (10d).

### 3. Filtro de Tendencia
- Si se activa el MA200:
  - Solo LONG cuando `close > MA200`
  - Solo SHORT cuando `close < MA200`

### 4. Piramidación
- Añade posición cada `addStepN × N` a favor de la tendencia.  
- Máximo de `maxUnits` acumuladas.

### 5. Tamaño de Posición
Qty = (Cuenta × Riesgo%) / (StopMultN × ATR)

---

## 🧩 Interfaz Visual

### 🖥️ HUD Lateral
Panel informativo con todos los datos del sistema:

Turtle v2 — Long & Short
N(ATR25)=14.84 | Stop=2.5N | Add=0.75N
Unidades=3 / 3 | Qty=0.40
Entrada=55d (última fue ganadora)
OK Long=Sí | OK Short=No


### 🟩 Semáforo Central
Validación de contexto, tendencia y señal:

| Fila | Significado | Verde | Rojo |
|------|--------------|-------|------|
| Contexto | Fecha y sesión válidas | ✅ | ❌ |
| Tendencia | A favor de MA200 | ✅ | ❌ |
| Señal | Ruptura activa | ✅ | ❌ |
| OK | Condiciones completas | ✅ | ❌ |

---

## ⚡ Alertas Automáticas

| Tipo | Condición | Mensaje |
|------|------------|----------|
| LONG Entry | Ruptura superior | `Turtle v2: LONG entry signal (breakout)` |
| SHORT Entry | Ruptura inferior | `Turtle v2: SHORT entry signal (breakdown)` |
| ADD Long | Nivel de piramidación alcanzado | `Turtle v2: ADD LONG level reached` |
| ADD Short | Nivel de piramidación alcanzado | `Turtle v2: ADD SHORT level reached` |

---

## 🧮 Parámetros Recomendados — Criptomonedas y Oro

| Activo | ATR (N) | Stop (xN) | Add (xN) | Riesgo % | MA Filter | Timeframe Sugerido |
|---------|----------|-----------|-----------|-----------|------------|--------------------|
| BTCUSDT | 25 | 2.5 | 0.75 | 0.5 | ✅ | 1D |
| ETHUSDT | 25 | 2.0 | 0.75 | 0.5 | ✅ | 1D |
| BNBUSDT | 30 | 2.5 | 0.75 | 0.5 | ✅ | 1D |
| SOLUSDT | 20 | 2.5 | 0.75 | 0.75 | ✅ | 4H |
| XRPUSDT | 25 | 2.0 | 0.5 | 0.5 | ✅ | 1D |
| ADAUSDT | 25 | 2.5 | 0.75 | 0.5 | ✅ | 1D |
| DOGEUSDT | 25 | 2.5 | 0.75 | 0.5 | ✅ | 4H |
| AVAXUSDT | 20 | 2.0 | 0.75 | 0.5 | ✅ | 1D |
| MATICUSDT | 25 | 2.0 | 0.75 | 0.5 | ✅ | 1D |
| DOTUSDT | 25 | 2.5 | 0.75 | 0.5 | ✅ | 1D |
| LINKUSDT | 25 | 2.0 | 0.75 | 0.5 | ✅ | 1D |
| LTCUSDT | 25 | 2.5 | 0.75 | 0.5 | ✅ | 1D |
| TRXUSDT | 25 | 2.0 | 0.75 | 0.5 | ✅ | 1D |
| TONUSDT | 20 | 2.0 | 0.5 | 0.5 | ✅ | 4H |
| SHIBUSDT | 25 | 2.5 | 0.75 | 0.75 | ✅ | 4H |
| XAUUSD | 35 | 2.0 | 0.5 | 0.3 | ✅ | 1D |

---

## 🧠 Estrategia Visual

📈 **Indicadores Clave:**
- **Canales Donchian (20, 55, 10)**  
- **Media móvil 200 periodos**  
- **ATR dinámico (25)**  

🟩 Verde = Señal válida  
🟥 Rojo = Condición bloqueada  

---

## 🚀 Uso Profesional

### 🔬 Modo Backtesting
1. Abre TradingView → *Pine Editor*  
2. Pega el código completo  
3. Selecciona el par y timeframe  
4. Ejecuta el *Strategy Tester*

### ⚙️ Modo Live Trading
- Activa `enableAlerts = true`  
- Configura webhooks hacia tu bot (n8n / 3Commas / Binance API)  
- Monitorea en tiempo real las señales con el semáforo y HUD.

---

## 🏛️ Créditos

Desarrollado por **Fernando Cuéllar**  
Data Scientist | Quant Developer | Fundador de [KuellarFer Labs](https://kuellarfer.com)

📧 **Correo:** [kuellarfer@gmail.com](mailto:kuellarfer@gmail.com)  
🐙 **GitHub:** [https://github.com/fercuellar](https://github.com/fercuellar)  
📊 **TradingView:** [FerCuellar](https://www.tradingview.com/u/FerCuellar)

---

## 📜 Licencia

```text
This Pine Script® code is subject to the terms of the Mozilla Public License 2.0.
© 2025 Fernando Cuéllar — All rights reserved.
