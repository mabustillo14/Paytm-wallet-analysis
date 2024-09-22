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
Es un archivo CSV de 16 columnas de información, contiene 5000 registros sintéticos de varias transacciones financieras en múltiples categorías, lo que proporciona una rica fuente para el análisis de tendencias y comportamientos de pago digital.

A continuación, se describen las principales entidades:
- **idx:** Índice único para cada registro
- **transaction_id:** Identificador único para cada transacción (UUID)
- **user_id:** Identificador único para cada usuario
- **transaction_date:** Fecha y hora de la transacción
- **product_category:** Categoría del producto o servicio
- **product_name:** Nombre específico del producto o servicio
- **merchant_name:** Nombre del comerciante o proveedor de servicios
- **product_amount:** Monto de la transacción en moneda local
- **transaction_fee:** Tarifa cobrada por la transacción
- **cashback:** Monto del reembolso recibido por la transacción
- **loyalty_points:** Puntos de fidelidad obtenidos por la transacción
- **payment_method:** Método utilizado para el pago
- **transaction_status:** Estado de la transacción (Successful, Failed, Pending)
- **merchant_id:** Identificador único para cada comerciante
- **device_type:** Tipo de dispositivo utilizado para la transacción
- **location:** Categoría de ubicación de la transacción







