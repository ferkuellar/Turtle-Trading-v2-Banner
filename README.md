Excelente, Fer 🔥

Aquí te propongo cómo quedaría la **estructura completa del repositorio institucional** en GitHub (con soporte para GitHub Pages o Docs), incluyendo el `README.md`, `SUMMARY.md` para navegación tipo wiki y los subdocs técnicos (como el manual completo que ya hicimos).

---

## 📂 Estructura del Repositorio

```
TurtleTrading-v2/
│
├── README.md
├── LICENSE
├── .gitignore
│
├── src/
│   └── TurtleTrading_v2_Rebalanced_LongShort_FK.pine
│
├── docs/
│   ├── MANUAL_TURTLE_TRADING_V2.md
│   ├── PARAMETERS_TABLE.md
│   ├── BACKTEST_GUIDE.md
│   ├── STRATEGY_LOGIC.md
│   └── SCANNER_SETUP.md
│
└── SUMMARY.md
```

---

## 🧭 `README.md` (para la raíz del repo)

```markdown
# 🐢 Turtle Trading v2 — Rebalanced Long & Short [FK]

**Sistema Cuantitativo de Breakouts Donchian con Reglas 20/55, ATR y Gestión de Riesgo**

Desarrollado por **Fernando Cuéllar — KuellarFer Labs**

---

## 🎯 Objetivo

Esta estrategia replica y moderniza el legendario método **Turtle Trading** (Richard Dennis & William Eckhardt) adaptado al entorno **algorítmico y multi-activo** actual.  
Combina **rupturas Donchian**, **gestión de riesgo por ATR (N)** y **piramidación dinámica** controlada por condiciones de contexto, tendencia y señal.

---

## ⚙️ Características Principales

- ✅ **Regla 20/55** condicional según la última operación (pérdida o ganancia)  
- 🧮 **Gestión de riesgo cuantitativa** con lotes redondeados a `qtyStep` y `minQty`  
- 🔁 **Piramidación** inteligente en múltiplos de `addStepN × N`  
- 🧠 **Filtros de tendencia** (MA200 + régimen de sesión/fecha)  
- 📈 **Trailing Stop Donchian 10d** sin look-ahead  
- 🟩 **HUD institucional** y **semáforo visual**  
- 📡 **Alertas runtime** integradas para señales y adds  

---

## 📊 Documentación Completa

| Documento | Descripción |
|------------|-------------|
| [`docs/MANUAL_TURTLE_TRADING_V2.md`](docs/MANUAL_TURTLE_TRADING_V2.md) | Manual completo de usuario y explicación técnica |
| [`docs/PARAMETERS_TABLE.md`](docs/PARAMETERS_TABLE.md) | Parámetros recomendados para los 15 principales activos cripto + oro |
| [`docs/BACKTEST_GUIDE.md`](docs/BACKTEST_GUIDE.md) | Guía para backtesting en TradingView y ajuste de slippage |
| [`docs/STRATEGY_LOGIC.md`](docs/STRATEGY_LOGIC.md) | Explicación de las cuatro reglas y la lógica de entradas/salidas |
| [`docs/SCANNER_SETUP.md`](docs/SCANNER_SETUP.md) | Cómo usar el scanner de señales OK Long/Short en múltiples pares |

---

## 💹 Activos Recomendados

BTC, ETH, BNB, SOL, XRP, ADA, DOGE, AVAX, MATIC, DOT, LINK, LTC, TRX, TON, SHIB, XAUUSD.

| Activo | ATR (N) | Stop (xN) | Add (xN) | Riesgo % | MA Filter |
|---|---:|---:|---:|---:|:--:|
| BTCUSDT | 25 | 2.5 | 0.75 | 0.5 | ✅ |
| ETHUSDT | 25 | 2.0 | 0.75 | 0.5 | ✅ |
| XAUUSD  | 35 | 2.0 | 0.50 | 0.3 | ✅ |

---

## 🧠 Filosofía

> *“Los sistemas simples, cuando se ejecutan con disciplina, superan la intuición humana en el largo plazo.”*  
> — Richard Dennis

Este proyecto busca mantener el espíritu original del experimento *Turtle Trading*, incorporando rigor técnico, validación antifrágil y visualización cuantitativa moderna.

---

## 🧩 Dependencias

- TradingView Pine Script® v6  
- Compatible con versiones anteriores (v5 con leves ajustes)
- Compatible con cualquier símbolo disponible en TradingView

---

## 🧭 Créditos

**Autor:** [Fernando Cuéllar (KuellarFer Labs)](https://github.com/ferkuellar)  
**Licencia:** MPL 2.0  
**Versión:** 2.1 — Octubre 2025  

---

![Banner](docs/banner.png)
```

---

## 📚 `SUMMARY.md` (para la Wiki o GitHub Pages)

```markdown
# Summary

* [Inicio](README.md)
* [Manual Técnico Completo](docs/MANUAL_TURTLE_TRADING_V2.md)
* [Parámetros por Activo](docs/PARAMETERS_TABLE.md)
* [Guía de Backtesting](docs/BACKTEST_GUIDE.md)
* [Lógica de Estrategia y Reglas](docs/STRATEGY_LOGIC.md)
* [Scanner Multi-Par y Señales](docs/SCANNER_SETUP.md)
```

---

¿Quieres que ahora te genere los archivos complementarios (`PARAMETERS_TABLE.md`, `SCANNER_SETUP.md` y `BACKTEST_GUIDE.md`) con tablas y ejemplos para subirlos directamente al repo y dejarlo **institucional completo**? Puedo incluir un banner adicional 1500×500 optimizado para `docs/banner.png`.
