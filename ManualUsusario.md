# 🐢 Turtle Trading v2 — Rebalanced Long & Short [FK]
**Manual de Usuario Técnico para GitHub (Pine Script® v6)**  
Autor: **Fernando Cuéllar** • Licencia: **MPL 2.0**

> Este documento es **institucional y exhaustivo**. Desmenuza el código línea por línea, explica cada regla, cada variable y cada decisión de diseño. Si algo se mueve en este script, aquí sabrás **por qué**.

---

## 0) Índice
1. [Objetivo y filosofía](#1-objetivo-y-filosofía)
2. [Arquitectura general del sistema](#2-arquitectura-general-del-sistema)
3. [Estructura del código](#3-estructura-del-código)
4. [Inputs y configuración](#4-inputs-y-configuración)
5. [Contexto, filtros y régimen de mercado](#5-contexto-filtros-y-régimen-de-mercado)
6. [Regla 20/55 según última operación](#6-regla-2055-según-última-operación)
7. [Donchian sin look-ahead](#7-donchian-sin-lookahead)
8. [ATR, riesgo y sizing](#8-atr-riesgo-y-sizing)
9. [Estado del trade y resets](#9-estado-del-trade-y-resets)
10. [Señales y validación de las 4 reglas](#10-señales-y-validación-de-las-4-reglas)
11. [Entradas iniciales (órdenes stop)](#11-entradas-iniciales-órdenes-stop)
12. [Piramidación (adds)](#12-piramidación-adds)
13. [Exits: stop inicial vs trailing Donchian](#13-exits-stop-inicial-vs-trailing-donchian)
14. [Visualización: shapes, semáforo, HUD, fondos](#14-visualización-shapes-semáforo-hud-fondos)
15. [Alertas runtime](#15-alertas-runtime)
16. [Parámetros recomendados por activo](#16-parámetros-recomendados-por-activo)
17. [Rendimiento, precisión y backtesting](#17-rendimiento-precisión-y-backtesting)
18. [Extensiones sugeridas](#18-extensiones-sugeridas)
19. [Solución de problemas](#19-solución-de-problemas)
20. [Apéndice: fórmulas y pseudocódigo](#20-apéndice-fórmulas-y-pseudocódigo)
21. [Licencia y créditos](#21-licencia-y-créditos)

---

## 1) Objetivo y filosofía
**Turtle v2 [FK]** implementa un **sistema de ruptura Donchian** con **regla 20/55**, **piramidación por múltiplos de N (ATR)**, **gestión de riesgo cuantitativa**, y **filtros de tendencia**.  
Principio: *no predecir*; **seguir** la tendencia, **controlar** el riesgo, **dejar correr** beneficios.

---

## 2) Arquitectura general del sistema
- **Inputs** → parámetros operativos (riesgo, filtros, reglas).
- **Contexto** → fecha/sesión + filtro MA para régimen alcista/bajista.
- **Regla 20/55** → longitud de Donchian condicionada al resultado de la última operación.
- **Señales** → ruptura del canal (sin look-ahead).
- **Sizing** → riesgo fijo en USD / (stop en precio = `stopMultN × ATR`).
- **Entradas** → órdenes stop en el canal; **piramidación** por escalones de `addStepN × N`.
- **Exits** → `max/min` entre **stop inicial** y **trailing Donchian (10d)**.
- **Visual** → señales, semáforo, HUD, tintes de fondo.
- **Alertas** → eventos clave al cierre.

---

## 3) Estructura del código
Bloques principales del script:
1. **Helpers** (`max2`, `min2`)
2. **Inputs**
3. **Contexto & filtros**
4. **Regla 20/55** (tracking de P/L de la última operación)
5. **Donchian sin look-ahead** (`[1]`)
6. **ATR & sizing** (redondeo a `qtyStep` y `minQty`)
7. **Estado del trade**
8. **Señales**
9. **Entradas iniciales**
10. **Piramidación**
11. **Exits**
12. **Visualización (plots, shapes, semáforo, HUD)**
13. **Alertas**

---

## 4) Inputs y configuración

### 4.1 Modo / Reglas
- `use20OnlyAfterLoss (bool)`  
  Si la última operación fue pérdida → **usa 20d**; si fue ganancia → **55d**.
- `directionMode (Both | Long only | Short only)`  
  Habilita lados de operación.

### 4.2 Sizing / Riesgo
- `risk_pct (float)`  
  % de la “cuenta base” arriesgado por operación.
- `account (float)`  
  Base nominal para sizing (puede diferir del equity real para reproducibilidad).

### 4.3 Filtros
- `useMAFilter (bool)` + `maLen` (por defecto 200).  
  Activa filtro de tendencia **MA200**.
- `dateFrom (time)` + `sessionStr (session)`  
  Ventana temporal y sesión válida (por exchange).

### 4.4 Avanzado
- `atrLen (int)` → N (ATR de Wilder por `ta.atr`).
- `stopMultN (float)` → stop inicial = `stopMultN × N`.
- `addStepN (float)` → paso entre adds = `addStepN × N`.
- `maxUnits (int)` → límite de unidades acumuladas.

### 4.5 Símbolo (tamaño mínimo)
- `qtyStep`, `minQty` → ***compatibilidad real*** con lotes/contratos del broker.

### 4.6 Alertas
- `enableAlerts (bool)` → activa `alert()` en eventos relevantes.

> **Diseño**: todos los inputs están **namespaced** por grupos (`group=...`) para un panel limpio.

---

## 5) Contexto, filtros y régimen de mercado
```pine
inDate = time >= dateFrom
inSess = not na(time(timeframe.period, sessionStr))
ma     = ta.sma(close, maLen)

allowLong  = directionMode == "Both" or directionMode == "Long only"
allowShort = directionMode == "Both" or directionMode == "Short only"

trendLongOK  = not useMAFilter or close > ma
trendShortOK = not useMAFilter or close < ma
````

* **Contexto**: evita operar fuera de fecha/sesión.
* **Régimen**: si el **filtro MA** está ON → exige `close > MA` para **longs**, `close < MA` para **shorts**.
* **Dirección**: `directionMode` acota lados.

---

## 6) Regla 20/55 según última operación

```pine
var float prevNet = na
netNow      = strategy.netprofit
tradeClosed = not na(prevNet) and netNow != prevNet
lastPL      = tradeClosed ? (netNow - prevNet) : 0.0
var bool lastTradeLoss = true
if barstate.isconfirmed
    prevNet := netNow
if tradeClosed
    lastTradeLoss := (lastPL < 0)

entry20 = 20
entry55 = 55
exitLen = 10
use55        = use20OnlyAfterLoss and not lastTradeLoss
effEntryLen  = use55 ? entry55 : entry20
```

* **Seguimiento de P/L**: cuando `strategy.netprofit` cambia entre barras, **una operación cerró**.
* **Regla**: si la última operación fue **ganadora** → **55d**; si fue **perdedora** → **20d** (cuando el toggle está activo).
* **Motivo**: replica el comportamiento clásico de *Turtles* (Sistema 1 / Sistema 2).

---

## 7) Donchian sin look-ahead

```pine
upperEntry = ta.highest(high, effEntryLen)[1]  // breakout long
lowerEntry = ta.lowest(low,  effEntryLen)[1]   // breakdown short
upperExit  = ta.highest(high, exitLen)[1]      // trailing short
lowerExit  = ta.lowest(low,  exitLen)[1]       // trailing long
```

* **`[1]` es crucial**: evita incluir la vela actual en el cálculo → **sin sesgo de anticipación**.
* **Entradas**: breakout de **upperEntry** / breakdown de **lowerEntry**.
* **Exits**: trailing por Donchian 10 días, espejo al lado opuesto.

---

## 8) ATR, riesgo y sizing

```pine
N       = ta.atr(atrLen)
riskUSD = account * (risk_pct / 100.0)
rawQty  = N > 0 ? (riskUSD / (stopMultN * N)) : 0.0

lotstep = qtyStep
minlot  = max2(minQty, lotstep)

roundQty(q) =>
    qRounded = math.round(q / lotstep) * lotstep
    max2(qRounded, minlot)

qtyPerUnit = roundQty(rawQty)
```

* **Fundamento**:
  [
  \text{qty} ;=; \frac{$,\text{riesgo}}{\text{distancia stop}} ;=; \frac{(\text{account}\cdot \text{risk_%})}{(\text{stopMultN}\cdot N)}
  ]
* **Industrialización**: `roundQty()` **redondea** por `qtyStep` y respeta `minQty` → compatible con broker/derivados.
* **Consecuencia**: sizing **constante en USD** (no compounding) salvo que cambies `account` dinámicamente.

> **Tip**: para *compounding*, usa `strategy.equity` en lugar de `account`.

---

## 9) Estado del trade y resets

```pine
var float tradeN       = na
var float nextAddPrice = na
var int   unitsAdded   = 0

isLong  = strategy.position_size > 0
isShort = strategy.position_size < 0
isFlat  = strategy.position_size == 0

if isFlat
    unitsAdded   := 0
    nextAddPrice := na
    tradeN       := na
```

* **Persistencia** (`var`) de N al inicio del trade (**`tradeN := N[1]`**), nivel de próximo **add** y contador de **unidades**.
* **Reset** seguro cuando la posición está **flat**.

---

## 10) Señales y validación de las 4 reglas

```pine
longSignal  = inDate and inSess and allowLong  and trendLongOK  and not na(upperEntry) and high > upperEntry
shortSignal = inDate and inSess and allowShort and trendShortOK and not na(lowerEntry) and low  < lowerEntry

// Semáforo (Long & Short) + fondo
condContext     = inDate and inSess
condLongTrend   = trendLongOK  and allowLong
condShortTrend  = trendShortOK and allowShort
condLongSignal  = not na(upperEntry) and high > upperEntry and qtyPerUnit > 0
condShortSignal = not na(lowerEntry) and low  < lowerEntry and qtyPerUnit > 0

okLong  = condContext and condLongTrend  and condLongSignal
okShort = condContext and condShortTrend and condShortSignal
```

**Las 4 reglas (OK Long/Short):**

1. **Contexto**: fecha + sesión válidas
2. **Tendencia**: filtro MA200 (si ON)
3. **Señal**: ruptura Donchian vigente + `qtyPerUnit>0`
4. **OK**: conjunción de las tres

---

## 11) Entradas iniciales (órdenes stop)

```pine
if isFlat and longSignal
    strategy.entry("L-1", strategy.long,  stop=upperEntry, qty=qtyPerUnit)
    tradeN       := N[1]
    unitsAdded   := 1
    nextAddPrice := upperEntry + addStepN * tradeN

if isFlat and shortSignal
    strategy.entry("S-1", strategy.short, stop=lowerEntry, qty=qtyPerUnit)
    tradeN       := N[1]
    unitsAdded   := 1
    nextAddPrice := lowerEntry - addStepN * tradeN
```

* **Orden stop** en el **nivel de ruptura**.
* **`tradeN`** toma `N[1]` (N de la barra previa al fill) → establece una **escala fija** de adds por trade.

---

## 12) Piramidación (adds)

```pine
posUnits = qtyPerUnit > 0 ? math.round(math.abs(strategy.position_size) / qtyPerUnit) : 0

if isLong and unitsAdded < maxUnits and posUnits == unitsAdded and not na(nextAddPrice) and tradeN > 0 and close >= nextAddPrice
    strategy.entry("L-" + str.tostring(unitsAdded + 1), strategy.long, qty=qtyPerUnit)
    unitsAdded   += 1
    nextAddPrice += addStepN * tradeN

if isShort and unitsAdded < maxUnits and posUnits == unitsAdded and not na(nextAddPrice) and tradeN > 0 and close <= nextAddPrice
    strategy.entry("S-" + str.tostring(unitsAdded + 1), strategy.short, qty=qtyPerUnit)
    unitsAdded   += 1
    nextAddPrice -= addStepN * tradeN
```

* **Condición de seguridad**: `posUnits == unitsAdded` evita **dobles fills** si la ejecución parcial redondea cantidades.
* **Escalón**: cada add se **separa** `addStepN × tradeN` del nivel anterior (clásico Turtle: 0.5–1.0 N).
* **Límite**: `maxUnits` incluye la **unidad inicial**.

---

## 13) Exits: stop inicial vs trailing Donchian

```pine
useN = nz(tradeN, N)

if isLong
    stopInitL  = strategy.position_avg_price - stopMultN * useN
    stopTrailL = lowerExit
    stopLong   = max2(stopInitL, stopTrailL)
    strategy.exit("XL", from_entry="L-1", stop=stopLong)

if isShort
    stopInitS  = strategy.position_avg_price + stopMultN * useN
    stopTrailS = upperExit
    stopShort  = min2(stopInitS, stopTrailS)
    strategy.exit("XS", from_entry="S-1", stop=stopShort)
```

* **`useN`**: si `tradeN` no existe (edge cases), cae a `N` actual.
* **Racional**: en **largos**, el stop efectivo es el **máximo** entre (stop inicial por N) y el **Donchian inferior (10d)** → **nunca afloja** el stop.
  Espejo para **shorts** con `min2`.

---

## 14) Visualización: shapes, semáforo, HUD, fondos

* **Shapes**:

  * Triángulos `LONG/SHORT` cuando hay **señal** y estás **flat**.
  * Círculos **ADD** cuando el precio cruza `nextAddPrice`.
* **Semáforo central (tabla)**:

  * Filas: *Padding*, **LONG/SHORT**, *Contexto*, *Tendencia*, *Señal*, **OK**.
  * Colores con **fade** (alpha alto cuando no cumple).
* **HUD lateral**:

  * Posicionado a **10 barras** a la derecha y **0.5N** sobre el `high`.
  * Muestra **ATR, StopN, AddN, Unidades, Qty**, regla 20/55 activa y **OK Long/Short**.
* **Tintes de fondo**:

  * Verde/rojo suave cuando **okLong/okShort** están activos en la barra.

> **Anti-flicker**: se actualiza **con `barstate.isnew`** (una vez por barra).

---

## 15) Alertas runtime

```pine
send(msg) =>
    alert(msg, alert.freq_once_per_bar_close)

// Dispara mensajes SOLO al cierre de vela
if enableAlerts and barstate.isconfirmed
    if isFlat and longSignal
        send("Turtle v2: LONG entry signal (breakout)")
    if isFlat and shortSignal
        send("Turtle v2: SHORT entry signal (breakdown)")
    if isLong and not na(nextAddPrice) and close >= nextAddPrice
        send("Turtle v2: ADD LONG level reached")
    if isShort and not na(nextAddPrice) and close <= nextAddPrice
        send("Turtle v2: ADD SHORT level reached")
```

* **Frecuencia**: **una por cierre de vela** (*no* spam intra-bar).
* **Casos**: señales de entrada y niveles de **piramidación**.

---

## 16) Parámetros recomendados por activo

*(1D sugerido; ajustar en 4H con factores ~0.6–0.7)*

| Activo    | ATR (N) | Stop (xN) | Add (xN) | Riesgo % | MA Filter |
| --------- | ------: | --------: | -------: | -------: | :-------: |
| BTCUSDT   |      25 |       2.5 |     0.75 |      0.5 |     ✅     |
| ETHUSDT   |      25 |       2.0 |     0.75 |      0.5 |     ✅     |
| BNBUSDT   |      30 |       2.5 |     0.75 |      0.5 |     ✅     |
| SOLUSDT   |      20 |       2.5 |     0.75 |     0.75 |     ✅     |
| XRPUSDT   |      25 |       2.0 |     0.50 |      0.5 |     ✅     |
| ADAUSDT   |      25 |       2.5 |     0.75 |      0.5 |     ✅     |
| DOGEUSDT  |      25 |       2.5 |     0.75 |      0.5 |     ✅     |
| AVAXUSDT  |      20 |       2.0 |     0.75 |      0.5 |     ✅     |
| MATICUSDT |      25 |       2.0 |     0.75 |      0.5 |     ✅     |
| DOTUSDT   |      25 |       2.5 |     0.75 |      0.5 |     ✅     |
| LINKUSDT  |      25 |       2.0 |     0.75 |      0.5 |     ✅     |
| LTCUSDT   |      25 |       2.5 |     0.75 |      0.5 |     ✅     |
| TRXUSDT   |      25 |       2.0 |     0.75 |      0.5 |     ✅     |
| TONUSDT   |      20 |       2.0 |     0.50 |      0.5 |     ✅     |
| SHIBUSDT  |      25 |       2.5 |     0.75 |     0.75 |     ✅     |
| XAUUSD    |      35 |       2.0 |     0.50 |      0.3 |     ✅     |

> **TPs opcionales**: si deseas parciales, usa límites en `1.0/1.5/2.0 × N` y conserva el trailing Donchian como salida final.

---

## 17) Rendimiento, precisión y backtesting

* `calc_on_every_tick=true` → ejecuciones intra-bar más fieles; el backtest **puede diferir** de close-only.
* **Sin look-ahead**: todos los Donchian usan `[1]`.
* **Ejecución**: `strategy.entry(..., stop=...)` simula **stop orders**, no market instantáneo.
* **Slippage/comisiones**: configúralos en las **propiedades** de la strategy.
* **Lotes reales**: `qtyStep/minQty` evita rechazos por tamaños ilegales.
* **Sensibilidad**: `atrLen`, `stopMultN` y `addStepN` afectan el perfil riesgo/beneficio.

---

## 18) Extensiones sugeridas

* **Riesgo sobre equity**: `riskUSD = strategy.equity * (risk_pct/100)`.
* **TPs parciales** configurables (1–3) con `qty_percent` (manteniendo trailing).
* **Filtro adicional** (RSI/ADX/Volumen) para evitar breakouts débiles.
* **Panel de métricas** (p. ej., profit factor rolling, % adds usados).
* **Scanner externo** (indicador) para detectar **OK Long/Short** multi-símbolo.
* **Parámetros por símbolo** (switch por `syminfo.tickerid`) para preajustes institucionales.

---

## 19) Solución de problemas

| Síntoma                 | Causa probable                                      | Solución                                                              |
| ----------------------- | --------------------------------------------------- | --------------------------------------------------------------------- |
| No ejecuta entradas     | `qtyPerUnit` < `minQty` o `qtyStep` mal configurado | Ajusta `qtyStep/minQty` al símbolo; verifica `rawQty` y redondeo      |
| ADDs duplicados         | Redondeos del broker o fills parciales              | La condición `posUnits == unitsAdded` ya lo mitiga; revisar `qtyStep` |
| HUD tapa velas          | HUD muy cerca del precio                            | Cambia `bar_index + 10` por `+15/+20` o usa `close + N*0.25`          |
| Señales “desaparecen”   | Filtro MA200 bloquea                                | Desactívalo temporalmente o revisa régimen (precio vs MA)             |
| Entradas tardías        | Donchian con `[1]` evita look-ahead                 | Correcto; es deliberado para precisión de backtest                    |
| Diferencias en backtest | `calc_on_every_tick=true` y slippage                | Ajustar slippage/comisiones; probar close-only para contraste         |
| Exceso de adds          | `addStepN` muy bajo o `maxUnits` alto               | Subir `addStepN` o bajar `maxUnits`                                   |
| Riesgo inconsistente    | `account` fijo y no equity                          | Cambiar a `strategy.equity` para compounding                          |

---

## 20) Apéndice: fórmulas y pseudocódigo

### 20.1 Fórmulas

* **DonchianUpper**(_L) = `highest(high, L)[1]`
* **DonchianLower**(_L) = `lowest(low, L)[1]`
* **Stop inicial (long)** = `avg_price − stopMultN × N`
* **Stop trailing (long)** = `lowerExit` (Donchian 10d)
* **Stop efectivo (long)** = `max(stopInicial, stopTrailing)`
* **Qty por unidad** = (\dfrac{\text{account} \cdot \text{risk_%}}{\text{stopMultN} \cdot N}) redondeado a `qtyStep`, ≥ `minQty`.

### 20.2 Pseudocódigo (flujo)

```text
calc contexto (fecha/sesión/MA) → elegir L = 20 o 55 según última P/L
Donchian(L) y Donchian(10)  // [1] evita look-ahead
N = ATR(atrLen)
qty = round( (account*risk%) / (stopMultN*N) )

if flat and breakout(upper) and filtros OK:
    entry stop @upper; tradeN=N[1]; units=1; nextAdd=upper + addStepN*tradeN
elif flat and breakdown(lower) and filtros OK:
    entry stop @lower; tradeN=N[1]; units=1; nextAdd=lower - addStepN*tradeN

if long and close >= nextAdd and units<maxUnits and posUnits==units:
    add long qty; units++; nextAdd += addStepN*tradeN
// espejo para short

stopLong = max( avg - stopMultN*useN , lowerExit )
stopShort= min( avg + stopMultN*useN , upperExit )
```

---

## 21) Licencia y créditos

**Licencia:** Mozilla Public License 2.0
**© Fernando Cuéllar — KuellarFer Labs**

Este proyecto implementa una reinterpretación moderna del sistema **Turtle Trading** con prácticas **cuantitativas e institucionales**.
Contribuciones, issues y PRs son bienvenidos.

