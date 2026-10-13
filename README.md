# 📱 Análisis de comportamiento de clientes — ConnectaTel

Análisis exploratorio y segmentación de clientes de una empresa de telecomunicaciones con operaciones en **México y Colombia**. A partir de tres fuentes de datos (planes, usuarios y uso real) se limpian los datos, se construye un perfil estadístico de uso por cliente, se detectan comportamientos atípicos y se segmenta la base por **edad** y **nivel de uso**, para proponer mejoras a la oferta comercial.

> Proyecto del Sprint 7 del programa de Analista de Datos de TripleTen.

---

## 🎯 Objetivo y preguntas de negocio

- ¿Qué segmentos de clientes muestran mayor o menor uso de llamadas y mensajes?
- ¿Qué usuarios presentan valores atípicos que puedan indicar comportamientos inusuales, fraude o errores de registro?
- ¿Cómo varía el uso según la edad y el tipo de plan contratado?
- ¿Qué patrones pueden ayudar a diseñar mejores planes y mejorar la satisfacción del cliente?

## 🗂️ Datos

| Archivo | Filas × columnas | Contenido |
|---|---|---|
| `plans.csv` | 2 × 8 | Catálogo de planes (Básico y Premium): mensajes, minutos y GB incluidos, pago mensual y costo por extra |
| `users_latam.csv` | 4.000 × 8 | `user_id`, nombre, edad, ciudad, fecha de registro, plan y fecha de baja (`churn_date`) |
| `usage.csv` | 40.000 × 6 | Registros de actividad en 2024: tipo (`call` / `text`), fecha, duración (llamadas) y longitud (mensajes) |

Los archivos provienen de la plataforma de TripleTen y **no se incluyen en este repositorio**. Para reproducir el análisis, colócalos en la ruta `/datasets/` (o ajusta las rutas en la primera celda del notebook).

## 🧪 Metodología

1. **Carga y exploración:** estructura, tipos de datos y valores nulos de las tres tablas.
2. **Calidad de datos:** nulos, *sentinels*, fechas imposibles y validación de nulos estructurales.
3. **Limpieza:** reemplazo de sentinels y marcado de fechas fuera de rango.
4. **Perfil por usuario:** agregación de `usage` (mensajes, llamadas, minutos, caracteres) y unión con `users`.
5. **Distribuciones y outliers:** histogramas, boxplots y límites por método IQR.
6. **Segmentación:** `grupo_uso` y `grupo_edad` con `np.select`.
7. **Insights ejecutivos y recomendaciones.**

## 🔍 Problemas de calidad detectados y decisiones

| Problema | Magnitud | Decisión |
|---|---|---|
| `city` nula o con sentinel `"?"` | 469 nulos (11,7 %) + sentinels `"?"`; tras convertirlos a nulos quedan 565 (14,1 %) | Mantener como nulos; impide asignar país a esos clientes |
| `churn_date` nula | 3.534 filas (88,35 %) | **No es un error:** indica cliente activo. Se conserva y se deriva la bandera `churn` (466 clientes, 11,65 %) |
| `duration` / `length` nulas en `usage` | 22.076 y 17.896 filas | Nulos estructurales: la duración solo aplica a llamadas y la longitud solo a mensajes. Se dejan como NaN para no sesgar promedios |
| `date` nula en `usage` | 50 filas (0,125 %) | Proporción mínima; no es posible imputar una fecha de forma confiable |
| `reg_date` con año 2026 | 40 filas (1 %) | Año imposible (los datos llegan hasta 2024). Se marcan como `NaT` |
| Outliers | Duración hasta 155,69 (límite IQR 61,86); mensajes hasta 17 (límite 11,5); llamadas hasta 15 (límite 10,5) | **Se conservan:** son valores plausibles y el objetivo es justamente identificar a estos clientes. Se usa la mediana como medida de resumen |

## 📊 Hallazgos clave

**Perfil de uso por cliente (mediana):** 4 llamadas, 5 mensajes y ~19,8 minutos de duración. La distribución de las tres variables es unimodal con sesgo positivo; la duración es la más asimétrica y muestra un grupo aislado entre ~130 y ~155 minutos.

**Segmentos por edad** (Joven < 30, Adulto 30–59, Adulto Mayor ≥ 60)

| Grupo | % de la base | Llamadas | Mensajes | Duración | Alto uso | Bajo uso |
|---|---|---|---|---|---|---|
| Joven | 19,0 % | 4 | 5 | 19,2 | 6,7 % | 20,7 % |
| Adulto | 50,4 % | 4 | 5 | 20,0 | 7,6 % | 18,0 % |
| Adulto Mayor | 30,6 % | 4 | 5 | 19,7 | 6,1 % | 21,0 % |

➡️ **La edad no diferencia el consumo.** Las medianas son prácticamente idénticas entre grupos.

**Segmentos por nivel de uso**

| Grupo | % de la base | Mediana llamadas / mensajes / minutos | Churn |
|---|---|---|---|
| Bajo uso | ~19,4 % | 3 / 3 / 12,8 | 11,4 % |
| Uso medio | ~73,6 % | 5 / 6 / 21,6 | 11,5 % |
| Alto uso | ~7,0 % | 5 / 10 / 25,9 | 14,0 % |

- **Alto uso se explica por mensajería** (mediana de 10 mensajes), más que por llamadas.
- **Básico y Premium tienen el mismo perfil de consumo** (Uso medio 73,1 % vs 74,4 %; Alto uso 7,2 % vs 6,6 %). Pagar más no se traduce en usar más.
- **El bajo consumo no predice el abandono.** El mayor churn está en Alto uso (14,0 %), pero con ~280 clientes es una señal a monitorear, no una conclusión firme.

## 💡 Recomendaciones

1. **Rediseñar la diferencia entre Básico y Premium** hacia valor percibido (datos, roaming, servicios adicionales), dado que el consumo es el mismo.
2. **Retener al segmento de Alto uso** con alertas al acercarse al límite del plan y beneficios personalizados.
3. **Crear un paquete de mensajería ampliada** para el perfil de Alto uso.
4. **Segmentar campañas por comportamiento, no por edad.**
5. **Validar las duraciones extremas (~130–155 min)** con fraude y operaciones y definir alertas automáticas; si son clientes legítimos de uso profesional, evaluar un plan empresarial.

## ⚠️ Limitaciones

- No se realizó el análisis por **país** (México / Colombia): la variable `city` no se transformó a país y tiene ~14 % de valores faltantes.
- `grupo_uso` se definió con conteos de llamadas y mensajes, no con duración; un cliente con pocas llamadas muy largas puede quedar subclasificado.
- No se contrastó el consumo real contra los límites de cada plan (`plans.csv`), por lo que la recomendación sobre Premium debe confirmarse con ese cruce.
- Los registros de `usage` cubren de enero a junio de 2024, mientras que los registros de clientes llegan hasta diciembre de 2024.
- El churn por grupo de uso es una comparación descriptiva; no se aplicaron pruebas estadísticas.

## 🛠️ Herramientas

Python · pandas · NumPy · seaborn · matplotlib · Jupyter Notebook

## ▶️ Cómo reproducir

1. Clona el repositorio:
   ```bash
   git clone <URL-DEL-REPOSITORIO>
   cd <NOMBRE-DEL-REPOSITORIO>
   ```
2. Instala las dependencias:
   ```bash
   pip install pandas numpy seaborn matplotlib jupyter
   ```
3. Coloca `plans.csv`, `users_latam.csv` y `usage.csv` en `/datasets/` (o cambia las rutas en la celda de carga).
4. Abre `telecom-analysis.ipynb` y ejecuta *Kernel → Restart & Run All*. También funciona en Google Colab subiendo los CSV al entorno.

## 📁 Estructura

```
├── telecom-analysis.ipynb   # Análisis completo
└── README.md
```
