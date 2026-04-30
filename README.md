=== HALLAZGOS DEL EDA SEMANA 1===

Dataset: 7,043 clientes | 21 variables | 26.5% tasa de churn

Variables con mayor señal predictiva:

1. Contract (tipo de contrato)
   - Month-to-month: ~43% de churn
   - One year:        ~11% de churn
   - Two year:         ~3% de churn
   → La variable más discriminante con diferencia

2. tenure (meses con la empresa)
   - Clientes con churn tienen en promedio 18 meses vs 37 meses los que se quedan
   → Los primeros meses son el período crítico de retención

3. OnlineSecurity y TechSupport
   - Sin servicio: ~42% churn | Con servicio: ~15% churn
   → Los servicios de valor agregado retienen clientes

4. MonthlyCharges
   - Media clientes que se van: $74.4 | Media clientes que quedan: $61.3
   → Cargos altos sin percepción de valor = mayor riesgo

5. InternetService
   - Fiber optic: ~42% churn (el más alto)
   - DSL: ~19% | Sin internet: ~7%
   → Posible problema de satisfacción con fibra óptica

Variable sin señal:
   - gender: ~26% en ambos grupos (descartar como feature)
Problema de datos:


=== DECISIONES DE FEATURE ENGINEERING SEMANA 2 ===

Variables creadas y su hipótesis de negocio:

  es_cliente_nuevo        → tenure <= 6 meses = período crítico de abandono
  cargo_por_mes           → MonthlyCharges / (tenure+1) = velocidad de gasto
  num_servicios           → suma de servicios activos = nivel de compromiso
  sin_servicios_extra     → flag sin ningún servicio adicional
  contrato_mensual        → flag Month-to-month = mayor libertad de salida
  cargo_sobre_media_contrato → paga más que el promedio de su tipo de contrato

Decisiones del Pipeline:

  Imputación numérica   → mediana (robusta ante outliers en MonthlyCharges)
  Imputación categórica → moda (valor más frecuente)
  Escalado              → StandardScaler (necesario para logistic regression)
  Encoding              → OneHotEncoder con handle_unknown='ignore'
                          (evita errores con categorías no vistas en producción)
  Split                 → 80/20 con stratify=y (mantiene proporción de churn)

Data leakage prevenido:
  → train_test_split ANTES de cualquier transformación
  → Toda la transformación ocurre dentro del Pipeline
  → El preprocessor se fitea SOLO en X_train
   - TotalCharges venía como string → convertido a numérico
   - 11 nulos en TotalCharges = clientes con tenure=0 → imputar con 0

Desbalance de clases: 73.5% / 26.5%
→  AUC-ROC y F1-score como métricas principales, NO accuracy

=== DECISIONES DE MODELADO — SEMANA 3 ===

¿Por qué empezar con Logistic Regression?
  → Es el baseline interpretable. Sus coeficientes dicen exactamente qué
    variable empuja hacia churn y cuánto. Si ya da un AUC decente,
    justifica la complejidad de un modelo de ensemble.

¿Por qué class_weight='balanced' en ambos modelos?
  → El dataset tiene ~26% de churn. Sin este ajuste, el modelo optimiza
    para la clase mayoritaria y pierde sensibilidad al churn.
    balanced pondera cada clase inversamente proporcional a su frecuencia.

¿Por qué max_depth=8 en Random Forest?
  → Sin límite de profundidad, los árboles crecen hasta memorizar
    el training set (overfitting). Restringir la profundidad actúa
    como regularización explícita.

¿Por qué AUC-ROC y no accuracy?
  → Un modelo que predice siempre No Churn tiene 73% de accuracy
    y 0% de recall sobre churners. AUC-ROC mide la capacidad
    discriminativa sin depender del threshold elegido.

¿Cuál modelo es mejor?
  → Depende: si necesitas explicar la decisión a negocio, LR.
    Si priorizas performance puro, RF. En semana 4 añadimos XGBoost
    y ajustamos el threshold para el costo real del negocio.


---
## Lo que aprendiste esta semana 4

**Aprendizajes:**
- `scale_pos_weight` en XGBoost penaliza más los errores sobre la clase minoritaria sin modificar los datos de entrenamiento
- SMOTE genera ejemplos sintéticos interpolando entre vecinos de la clase minoritaria; útil para desbalances extremos pero puede introducir ruido
- El threshold por defecto (0.5) es arbitrario y casi nunca es el óptimo
- El threshold óptimo se elige según el costo relativo de cada tipo de error en el negocio, no solo por F1
- Bajar el threshold aumenta el recall pero baja la precision, y viceversa

**Preguntas típicas de entrevista:**

*¿Qué es SMOTE y cuándo lo usarías?*  
→ Crea ejemplos sintéticos de la clase minoritaria interpolando entre instancias reales. Lo usaría cuando el desbalance es extremo (>10:1) y `scale_pos_weight` solo no es suficiente. Su riesgo es introducir ejemplos artificiales que no representan la distribución real.

*¿Por qué no dejar el threshold en 0.5?*  
→ Porque 0.5 optimiza accuracy implícitamente. En churn, perder un cliente cuesta mucho más que contactar innecesariamente a uno. Ajustar el threshold a ese costo real reduce el costo total del sistema, que es lo que al negocio le importa.

*¿Cuándo preferirías XGBoost sobre Random Forest?*  
→ XGBoost construye árboles secuencialmente corrigiendo los errores del anterior (boosting), lo que generalmente da mejor performance con menos árboles. Random Forest construye árboles en paralelo de forma independiente (bagging), es más robusto al overfitting y más fácil de paralelizar. Para datasets tabulares medianos, XGBoost suele ganar en métricas; para producción con restricciones de latencia, Random Forest puede ser preferible.