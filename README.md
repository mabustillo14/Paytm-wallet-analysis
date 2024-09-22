## <br>💰 Análisis de Transacciones en una Wallet Digital
Este proyecto tiene como objetivo analizar las transacciones realizadas en una billetera digital, utilizando diferentes métricas clave (KPIs) para obtener insights sobre el comportamiento de los usuarios, las transacciones y el rendimiento del sistema. 

### Caso de Estudio
Se utiliza un conjunto de datos obtenida de [Kaggle](https://www.kaggle.com/datasets/harunrai/digital-wallet-transactions) que simula las transacciones de una plataforma de billetera digital similar a servicios populares como PayTm en India o Khalti en Nepal.


## <br>💡 Enfoque Metodológico 
Se diseñaron dashboards interactivos en Power BI para visualizar KPIs y validar hipótesis relacionadas con cashback, lealtad, dispositivos y categorías de productos.

### Objetivos del Caso de Negocios:
- Analizar patrones de transacciones y comportamientos de usuarios.
- Explorar la relación entre cashback, lealtad y montos de transacción.
- Comparar el rendimiento en dispositivos móviles vs. computadoras.
- Identificar oportunidades de optimización a través de la tasa de conversión y abandono.

----

### Hipótesis de Negocio
1. Los usuarios que reciben más cashback tienden a realizar más transacciones.
2. Los usuarios que acumulan más puntos de lealtad son más propensos a realizar transacciones de mayor monto.
3. Los usuarios que usan dispositivos móviles tienen una tasa de conversión más baja que aquellos que usan computadoras
4. Las categorías de productos con mayor cashback tienen un monto promedio por transacción más alto

----

### Plan de Métricas
Para las pruebas de hipótesis de negocio, se plantean 13 KPIs que permitan examinar el comportamiento de los usuarios entre las distintas transacciones.

A continuación, se detallan las métricas utilizadas, junto con su cálculo y los puntos de vista desde los cuales se pueden analizar. Se puede ver con más detalles en la documentación del [Plan de Métricas](/docs/plan_metricas.pdf)

![Plan de métricas](/images/plan_metricas.JPG)


## <br>⚙️ Desarrollo del proyecto
### Descripción de la Fuente de Datos
Es un archivo CSV de 16 columnas de información. Contiene 5000 registros sintéticos de varias transacciones financieras en múltiples categorías, lo que proporciona una rica fuente para el análisis de tendencias y comportamientos de pago digital.

----
### Data Flow
A continuación se detalla el flujo de datos utilizado para la Extracción, Transformación y Carga de Datos (ETL) en el proyecto


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
Luego, ya podremos realizar la conexión de datos desde **Power BI > Bigquery** para transformar la información


## <br>💡 Data Mart
En el contexto de este proyecto, el Data Mart se diseñó específicamente para centralizar y organizar la información clave relacionada con las transacciones de una billetera digital, facilitando el análisis de datos mediante la segmentación y almacenamiento eficiente de los parámetros más relevantes. 

Previamente se deben limpiar los datos para evitar errores durante el análisis. Para el caso, los datos utilizados esta pre-procesados y limpios, de tal manera que no hay NULLs ni transacciones repetidas.

Luego para el modelado y normalización de datos, se crea un **Data Mart con 6 tablas de dimensiones**. Lo que permite agrupar los datos por categorías como transacciones (`fact_transactions`), métodos de pago (`dim_payment_method`), locaciones (`dim_location`), dispositivos (`dim_device`), categorías de productos (`dim_produts`) y fechas (`dim_calendar`).

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

Durante el diseño de todos los dashboards, se consideraron aspectos de UX y Data Storytelling para que la interpretación de las visualizaciones sea lo más amigable posible.

![Dashboard HOME](/images/dashboard_0.jpg)

![Dashboard General](/images/dashboard_1.jpg)

![Dashboard Puntos_Lealtad](/images/dashboard_2.jpg)

![Dashboard Tasa_Conversion](/images/dashboard_3.jpg)

![Dashboard Cashback](/images/dashboard_4.jpg)

![Dashboard Ganancias](/images/dashboard_5.jpg)


