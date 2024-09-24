## 💰 Análisis de Transacciones en una Wallet Digital
Este proyecto tiene como objetivo analizar las transacciones realizadas en una billetera digital, utilizando diferentes métricas clave (KPIs) para obtener insights sobre el comportamiento de los usuarios, las transacciones y el rendimiento del sistema. 

### Caso de Estudio
Se utiliza un conjunto de datos obtenida de [Kaggle](https://www.kaggle.com/datasets/harunrai/digital-wallet-transactions) que simula las transacciones de una plataforma de billetera digital similar a servicios populares como PayTm en India o Khalti en Nepal.

### Herramientas
- **[BigQuery](https://cloud.google.com/bigquery/docs/introduction?hl=es-419):** Para almacenar los datos y ejecutar consultas SQL para extraer los datos necesarios y realizar el análisis exploratorio.
- **[Power Query](https://learn.microsoft.com/es-es/power-query/power-query-what-is-power-query):** Fue utilizado para la limpieza y transformación de los datos antes de su análisis en Power BI.
- **[Power BI](https://www.microsoft.com/es-es/power-platform/products/power-bi?msockid=03ad65a8640c67e7386870ab65d8668e):** Fue para la creación de los dashboards interactivos que visualizan los resultados del análisis.
- **[Figma](https://www.figma.com/es-la/):** Para el diseño de elementos visuales para los dashboard y la documentación.


## <br>💡 Enfoque Metodológico 
Se diseñaron dashboards interactivos en Power BI para visualizar KPIs y validar hipótesis relacionadas con cashback, lealtad, dispositivos y categorías de productos.

### Objetivos del Caso de Negocios:
- Analizar patrones de transacciones y comportamientos de usuarios.
- Explorar la relación entre cashback, lealtad y montos de transacción.
- Comparar el rendimiento en dispositivos móviles vs. computadoras.
- Identificar oportunidades de optimización a través de la tasa de conversión y abandono.

----

### Hipótesis de Negocio
1. Los usuarios que acumulan más puntos de lealtad son más propensos a realizar transacciones de mayor monto.
2. Los meses con mayores tasas de conversión en general se correlacionan con un aumento en el uso de dispositivos móviles, indicando que la optimización para estos dispositivos es fundamental para maximizar las transacciones.
3. Las categorías de productos con mayor cashback tienen un ticket promedio más alto.
4. Los meses en los que se reportan altos márgenes de ganancia en categorías específicas son seguidos por un incremento en las ganancias generales, sugiriendo que la rentabilidad de estas categorías impulsa el crecimiento del negocio.

----

### Plan de Métricas
Para las pruebas de hipótesis de negocio, se plantean 13 KPIs que permitan examinar el comportamiento de los usuarios entre las distintas transacciones.

A continuación, se detallan las métricas utilizadas, junto con su cálculo y los puntos de vista desde los cuales se pueden analizar. Se puede ver con más detalles en la documentación del [Plan de Métricas](/docs/plan_metricas.pdf)

![Plan de métricas](/images/plan_metricas.JPG)


## <br>⚙️ Desarrollo del proyecto
### Descripción de la Fuente de Datos
Es un archivo CSV de 16 columnas de información. Contiene 5000 registros sintéticos de varias transacciones financieras en múltiples categorías, lo que proporciona una rica fuente para el análisis de tendencias y comportamientos de pago digital.

![Bronze Dataset](/images/dataset_transactions.JPG)

----
### Data Flow
A continuación se detalla el flujo de datos utilizado para la Extracción, Transformación y Carga de Datos (ETL) en el proyecto

![Data Flow](/images/data_flow.png)

Se consideraron los siguientes aspectos para la manipulación de la base datos:

0. Las consultas están diseñadas para ejecutarse en **Google Cloud Platform > BigQuery**, considerando la sintaxis específica de este entorno.
1. Para la creación de la **Capa Bronce**, se sube el archivo CSV como la tabla `dataset` al conjunto de datos `digital_wallet_transactions` dentro del proyecto denominado `bronze_cape`.
2. Para la creación de la **Capa Silver**, donde se encuentra la información ya transformada, se crea la tabla `dataset` en el conjunto de datos `transformdata` dentro del proyecto denominado `silver_cape`.

Para crear la tabla en la Capa Silver, donde solo se seleccionarán 12 columnas que nos interesan para nuestro estudio, se deben ejecutar la siguiente query:

```
CREATE OR REPLACE TABLE `silver-cape.transformdata.dataset` AS
SELECT
  data.transaction_id,
  data.user_id,
  data.transaction_date,
  data.product_category,
  data.product_amount,
  data.transaction_fee,
  data.cashback,
  data.loyalty_points,
  data.payment_method,
  data.transaction_status,
  data.device_type,
  data.location
FROM `bronze-cape.digital_wallet_transactions.dataset` AS data
```
![Silver Dataset](/images/dataset_transformdata.JPG)

Luego, ya podremos realizar la conexión de datos desde **Power BI > Bigquery** para transformar la información


## <br>💡 Data Mart
En el contexto de este proyecto, el Data Mart se diseñó específicamente para centralizar y organizar la información clave relacionada con las transacciones de una billetera digital, facilitando el análisis de datos mediante la segmentación y almacenamiento eficiente de los parámetros más relevantes. 

Previamente se deben limpiar los datos para evitar errores durante el análisis. Para el caso, los datos utilizados esta pre-procesados y limpios, de tal manera que no hay NULLs ni transacciones repetidas.

Luego para el modelado y normalización de datos, se crea un **Data Mart con 5 tablas de dimensiones y 1 tabla de hechos**. Lo que permite agrupar los datos por categorías como transacciones (`fact_transactions`), métodos de pago (`dim_payment_method`), locaciones (`dim_location`), dispositivos (`dim_device`), categorías de productos (`dim_produts`) y fechas (`dim_calendar`).

![Arquitectura data mart](/images/data_mart.JPG)

Luego desde Power Query se unifican las consultas para obtener las relaciones entre cada tabla.

----
### Métricas de Rendimiento
Para este proyecto, se utilizaron medidas DAX para calcular los KPIs que se describen en el plan de métricas. A continuación, se detallan los DAX para la creación de estas métricas en la tabla `Medidas`.

- Monto Total de Transacciones
```
total_number_transactions = COUNT(fact_transactions[transaction_id])
```
- Monto Promedio por Transacción
```
$_avg_amount_transaction = Medidas[$_total_transaction_amount] / Medidas[total_number_transactions]
```
- Ganancias Totales
```
$_total_transaction_fee = SUM(fact_transactions[transaction_fee])
```
- Margen de Ganancia
```
fee_rate_total_sales = [$_total_transaction_fee] / [$_total_transaction_amount]
```
- Cashback otorgado
```
$_total_cashback_amount = SUM(fact_transactions[cashback]) 
```
- Tasa de Cashback sobre Ventas Totales
```
cashback_rate_total_sales = [$_total_cashback_amount] / [$_total_transaction_amount]
```
- Número de Usuarios Activos
```
total_active_users = DISTINCTCOUNT(fact_transactions[user_id])
```
- Frecuencia de Transacciones por Usuario
```
transaction_frequency_per_user = [total_number_transactions] / [total_active_users]
```
- Total de Puntos de Lealtad
```
total_loyalty_points = SUM(fact_transactions[loyalty_points])
```
- Tasa de Acumulación de Puntos de Lealtad
```
loyalty_points_accumulation_rate = [total_loyalty_points] / [total_number_transactions]
```
- Número Total de Transacciones
```
total_number_transactions = COUNT(fact_transactions[transaction_id])
```
- Tasa de Conversión
```
conversion_rate = DIVIDE(
    COUNTROWS(
        FILTER(
            fact_transactions, 
            fact_transactions[transaction_status] = "Successful"
        )
    ),
    COUNTROWS(fact_transactions)
)
```
- Tasa de Abandono de Transacciones
```
transaction_abandonment_rate = DIVIDE(
    COUNTROWS(
        FILTER(
            fact_transactions, 
            fact_transactions[transaction_status] = "Failed" || fact_transactions[transaction_status] = "Pending"
        )
    ),
    COUNTROWS(fact_transactions)
)
```
## <br>💡 Análisis de Datos
El análisis de datos se llevó a cabo mediante un enfoque descriptivo y comparativo. Se desarrolló un reporte con 1 portada y 5 dashboards. En cada página se muestran aspectos representativos de las transacciones y visualizaciones puntuales para responder a cada una de las hipótesis de negocios y obtener insights relevantes.

Durante el diseño de todos los dashboards, se consideraron aspectos de UX/UI y Data Storytelling para que la interpretación de las visualizaciones sea lo más amigable posible.


> 👉Se puede visualizar el tablero completo desde el siguiente [enlace](https://mabustillo14.github.io/Paytm-wallet-analysis/site/index.html) 

![Dashboard HOME](/images/dashboard_0.jpg)

![Dashboard General](/images/dashboard_1.jpg)

![Dashboard Puntos_Lealtad](/images/dashboard_2.jpg)

![Dashboard Tasa_Conversion](/images/dashboard_3.jpg)

![Dashboard Cashback](/images/dashboard_4.jpg)

![Dashboard Ganancias](/images/dashboard_5.jpg)

## <br>🔍 Prueba de Hipótesis
1. **Los usuarios que acumulan más puntos de lealtad son más propensos a realizar transacciones de mayor monto.**

Se ha realizado un análisis del dashboard de Puntos de Lealtad, enfocándose en los gráficos que muestran el Ranking de transacciones por monto y el Ranking de transacciones por puntos de Lealtad.

![Visualización del Dashboard Puntos de Lealtad](/images/visualization_hn1_1.png)

Se observa que en las transacciones de mayor monto, la cantidad de puntos de lealtad acumulados por cada usuario oscila entre 38 y 995 puntos (flecha verde). Por otro lado, al examinar las transacciones de los usuarios con la mayor cantidad de puntos de lealtad (los primeros con 995 puntos), se encuentra que los montos transaccionados varían entre $333 y $9.816 (flecha amarilla).

Mientras que los usuarios que realizan transacciones de mayor monto tienden a tener un promedio moderado de puntos de lealtad, aquellos con más puntos no necesariamente realizan transacciones de alto valor. En base a los resultados, se sugiere que no existe una correlación directa en su totalidad entre el monto de las transacciones y la cantidad de puntos de lealtad acumulados por los usuarios. Si bien se cumple para algunas transacciones, no es un caso que se repite entre todos los casos. **_La hipótesis se considera falsa_**.
  

---
2. **Los meses con mayores tasas de conversión en general se correlacionan con un aumento en el uso de dispositivos móviles, indicando que la optimización para estos dispositivos es fundamental para maximizar las transacciones.**

Se ha analizado el dashboard de Tasa de Conversión, filtrando los datos de los tres meses con las mayores tasas de conversión hasta la fecha: Agosto 2023, Octubre 2023 y Julio 2024 (resaltado en amarillo).

![Visualización del Tasad de Conversión](/images/visualization_hn2_1.png)

En Agosto de 2023, se observa que los medios de pago con mayor tasa de conversión fueron las tarjetas de crédito (Credit Card) y las transferencias bancarias (Bank Transfer), además de que las transacciones web lograron la mayor conversión.

![Visualización del Tasad de Conversión](/images/visualization_hn2_2.png)
Durante Octubre de 2023, las tarjetas de débito (Debit Card) y las tarjetas de crédito (Credit Card) destacaron como los medios de pago con mayor tasa de conversión, manteniéndose las transacciones web como las más efectivas.

![Visualización del Tasad de Conversión](/images/visualization_hn2_3.png)

Finalmente, en Julio de 2024, las transferencias bancarias (Bank Transfer) y las tarjetas de débito (Debit Card) fueron los medios de pago más exitosos, y se destaca que la mayor conversión se registró a través de dispositivos iOS y Android.

![Visualización del Tasad de Conversión](/images/visualization_hn2_4.png)

Aunque las transacciones web dominaron en los meses anteriores, el notable incremento en la conversión de dispositivos móviles en julio de 2024 sugiere que optimizar la experiencia de usuario para estos dispositivos mejoró la tasa de conversión. Los datos analizados respaldan la hipótesis de que los meses con mayores tasas de conversión se correlacionan con un aumento en el uso de dispositivos móviles. **_La hipótesis se considera verdadera_**.

---
3. **Las categorías de productos con mayor cashback tienen un ticket promedio más alto**

Al analizar el dashboard de Cashback, hemos comparado el Cashback otorgado con el Ticket Promedio en diversas categorías de servicio.

![Visualización del Dashboard Cashback](/images/visualization_hn3_1.png)

Los datos revelan que las categorías de "Streaming Service" y "Education Fee" (resaltadas en rojo) presentan los mayores montos de Cashback, mientras que su Ticket Promedio es relativamente más bajo en comparación con otras categorías. En contraste, las categorías de "Internet Bill" y "Gas Bill" (resaltadas en verde) muestran un Ticket Promedio significativamente más alto, pero el Cashback por transacción es relativamente bajo.

Las categorías con mayor Cashback no necesariamente corresponden a montos transaccionados altos, lo que sugiere que los factores que influyen en el Cashback pueden ser independientes del Ticket Promedio. Por lo tanto, no podemos establecer una relación directa entre un alto Cashback y un Ticket Promedio elevado. **_La hipótesis se considera falsa_**.

---
4. **Los meses en los que se reportan altos márgenes de ganancia en categorías específicas son seguidos por un incremento en las ganancias generales, sugiriendo que la rentabilidad de estas categorías impulsa el crecimiento del negocio.**

Se ha analizado el dashboard de Ganancias, enfocándose en las categorías de productos con los mayores márgenes de ganancia: Education Fee (0,56%), Grocery Shopping (0,55%), Gift Card (0,55%), Bus Ticket (0,55%) y Streaming Service (0,54%). Además, se examinaron los meses con las mayores ganancias: Octubre de 2023, Diciembre de 2023, Marzo de 2024 y Mayo de 2024.

![Visualización del Ganancias](/images/visualization_hn4_1.png)

Se debe analizar en cuáles de las categorías se presentan los mismos picos que en los meses de mayor ganancia. A continuación, se dan a conocer los resultados de esta comparativa:

- **Streaming Service:** Los picos de ganancia solo coinciden con diciembre de 2023.
- **Bus Ticket:** No se observa coincidencia entre los picos de ganancia y los meses de mayor ganancia.
- **Gift Card:** Los picos de ganancia coinciden con octubre de 2023, diciembre de 2023 y mayo de 2024.
- **Grocery Shopping:** Se observa coincidencia en octubre de 2023, diciembre de 2023 y mayo de 2024.
- **Education Fee:** Los picos de ganancia coinciden con octubre de 2023, diciembre de 2023 y marzo de 2024.

Los datos analizados apoyan la hipótesis de que los meses con altos márgenes de ganancia en categorías específicas están correlacionados con incrementos en las ganancias generales. Las categorías de Gift Card, Grocery Shopping y Education Fee no solo muestran márgenes significativos, sino que también presentan picos de ganancia en varios de los meses más lucrativos. Por otro lado, la categoría de Bus Ticket no refleja esta relación.

Estos hallazgos sugieren que la rentabilidad de ciertas categorías impulsa el crecimiento del negocio. **_La hipótesis se considera verdadera_**.

## <br>📧 Contacto
Cualquier duda del proyecto, conectemos por [LinkedIn](https://www.linkedin.com/in/mario-bustillo/)
