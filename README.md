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
   → problema de satisfacción con fibra óptica
