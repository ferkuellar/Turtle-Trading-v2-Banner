# 📊 Parámetros de TP/SL — Turtle Trading v2 [FK]

**Versión Institucional de Gestión de Riesgo Paramétrico**  
Autor: Fernando Cuéllar — KuellarFer Labs  
Fecha: Octubre 2025  

---

## 🧠 Enfoque de Cálculo

Cada operación usa un **Stop Loss (SL)** y **Take Profit (TP)** expresados como múltiplos de `ATR(N)`.  
El sistema estándar usa:  
- `SL = 2.5 × ATR`  
- `TP = 5 × ATR`  
- `ADD = 0.75 × ATR`  
- Riesgo estimado = 0.5% del capital por operación

Sin embargo, para interpretación práctica se expresan también como **porcentajes del precio de entrada**, útiles al momento de ejecutar operaciones manuales o automatizadas.

\[
SL\% = \frac{SL_{USD}}{Precio_{Entrada}} \times 100, \quad TP\% = \frac{TP_{USD}}{Precio_{Entrada}} \times 100
\]

---

## ⚙️ Tabla General — Top 15 Criptos + Oro

| Activo | Timeframe | ATR(25) | Stop (xN) | TP (xN) | SL % | TP % | Riesgo/Beneficio | Add Step | Riesgo % Capital | Observaciones |
|:--|:--:|:--:|--:|--:|--:|--:|:--:|:--:|:--:|:--|
| **BTCUSDT** | 1D | 14.8 | 2.5 | 5.0 | **2.3%** | **4.6%** | 1:2 | 0.75N | 0.5% | Base institucional |
| **ETHUSDT** | 1D | 9.5 | 2.0 | 5.0 | **2.0%** | **5.0%** | 1:2.5 | 0.75N | 0.5% | Alta liquidez |
| **BNBUSDT** | 1D | 6.1 | 2.0 | 4.5 | **1.8%** | **4.1%** | 1:2.2 | 0.75N | 0.6% | Tendencias limpias |
| **SOLUSDT** | 1D | 3.2 | 2.5 | 5.5 | **2.1%** | **4.7%** | 1:2.2 | 0.75N | 0.5% | Volátil / oportunista |
| **XRPUSDT** | 1D | 0.12 | 3.0 | 6.0 | **3.2%** | **6.4%** | 1:2 | 0.75N | 0.5% | Requiere paciencia |
| **ADAUSDT** | 1D | 0.15 | 2.5 | 5.0 | **3.0%** | **6.0%** | 1:2 | 0.75N | 0.5% | Ideal en rango medio |
| **DOGEUSDT** | 1D | 0.002 | 2.0 | 4.5 | **2.2%** | **5.0%** | 1:2.3 | 0.75N | 0.5% | Impulsivo, ajustar TP |
| **AVAXUSDT** | 1D | 0.85 | 2.0 | 4.0 | **1.9%** | **3.8%** | 1:2 | 0.75N | 0.5% | Buen candidato multiadd |
| **MATICUSDT** | 1D | 0.07 | 2.0 | 4.5 | **2.5%** | **5.6%** | 1:2.2 | 0.75N | 0.5% | Alta correlación ETH |
| **DOTUSDT** | 1D | 0.15 | 2.5 | 5.5 | **2.8%** | **6.2%** | 1:2.2 | 0.75N | 0.5% | Tendencia limpia |
| **LINKUSDT** | 1D | 0.25 | 2.5 | 5.0 | **2.5%** | **5.0%** | 1:2 | 0.75N | 0.6% | Ruido bajo, top midcap |
| **LTCUSDT** | 1D | 1.1 | 2.5 | 5.5 | **2.1%** | **4.6%** | 1:2.2 | 0.75N | 0.5% | Se comporta como metal |
| **TRXUSDT** | 1D | 0.004 | 2.0 | 4.0 | **1.8%** | **3.6%** | 1:2 | 0.75N | 0.4% | Sesgo defensivo |
| **TONUSDT** | 1D | 0.25 | 2.5 | 5.0 | **3.2%** | **6.4%** | 1:2 | 0.75N | 0.5% | Promedio medio/volátil |
| **XAUUSD (Oro)** | 1D | 24.0 | 2.0 | 4.0 | **1.6%** | **3.2%** | 1:2 | 0.5N | 0.3% | Ideal para portafolio mixto |

---

## 📈 Reglas Operativas

1. **SL** → Calculado automáticamente como `Entry ± (StopMult × ATR)`  
2. **TP** → Calculado automáticamente como `Entry ± (TPMult × ATR)`  
3. **Adds** → Activos cada `addStepN × ATR` a favor de la posición  
4. **Trailing Stop Donchian (10)** → Reemplaza al SL si la tendencia se mantiene  
5. **OK Long/Short** → Solo se permite entrada si se cumplen las 4 reglas (Contexto, Tendencia, Señal, Validación)

---

## 🧮 Fórmula en pseudocódigo (referencia Python / n8n / Notion)

```python
ATR = 14.8        # ATR actual
entry = 64500.0   # Precio de entrada
stop_mult = 2.5
tp_mult = 5.0

SL = entry - stop_mult * ATR
TP = entry + tp_mult * ATR
SL_pct = (entry - SL) / entry * 100
TP_pct = (TP - entry) / entry * 100

print(SL, TP, SL_pct, TP_pct)
````

---

## 💡 Uso práctico (Checklist)

| Paso | Acción                        | Descripción                                   |
| ---- | ----------------------------- | --------------------------------------------- |
| 1️⃣  | Determinar ATR actual         | Usa `ta.atr(25)` o valor de volatilidad media |
| 2️⃣  | Insertar precio de entrada    | Precio actual o nivel Donchian                |
| 3️⃣  | Aplicar múltiplos (2.5× / 5×) | Calcula SL/TP absolutos                       |
| 4️⃣  | Convertir a %                 | `(TP−Entry)/Entry × 100`                      |
| 5️⃣  | Verificar ratio R:R           | 1:2 mínimo                                    |
| 6️⃣  | Confirmar “OK Long / Short”   | Si todas las condiciones están en verde       |

---

## 🧠 Recomendaciones de aplicación

* **BTC y ETH:** usar parámetros base institucional (2.5× / 5×).
* **Altcoins medianas:** ampliar SL a 3.0× ATR para reducir falsos breakouts.
* **Metales (XAUUSD):** preferir `2× / 4× ATR` con trailing activo.
* **Apalancamiento:** máximo x5 en futuros o derivativos.
* **Gestión de capital:** máximo 0.5% por operación abierta.

---

## 🔒 Nota de seguridad

> Este framework no busca predecir precios, sino **reaccionar con precisión cuantitativa a rupturas confirmadas**.
> Su propósito es institucional: controlar riesgo, estabilizar volatilidad y permitir reproducibilidad matemática.

---

📘 **Documento relacionado:** [Manual de Usuario Técnico (ManualUsuario.md)](ManualUsusario.md)




