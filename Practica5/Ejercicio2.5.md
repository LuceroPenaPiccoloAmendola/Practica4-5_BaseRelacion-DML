# Ejercicio 2.5: Integridad Referencial en el Almacén Asignado (Data Warehouse CDMX)

En este documento se analiza la estructura de modelado analítico (esquema en estrella) correspondiente al proyecto asignado sobre **Consumo de Agua en la Ciudad de México**, disponible en el repositorio de referencia [`LuceroPenaPiccoloAmendola/Data_Warehouse_static`](https://github.com/LuceroPenaPiccoloAmendola/Data_Warehouse_static).

## 1. Identificación del Modelo OLAP (Hechos, Dimensiones y Granularidad)

A partir de la inspección del repositorio y del esquema dimensional implementado en el data warehouse:

* **Repositorio de Origen:** [LuceroPenaPiccoloAmendola/Data_Warehouse_static](https://github.com/LuceroPenaPiccoloAmendola/Data_Warehouse_static)
* **Caso de Estudio:** Registro histórico del consumo de agua potable en la Ciudad de México por sectores y alcaldías.

### 1.1 Componentes del Almacén de Datos

1. **Tabla de Hechos (*Fact Table*): `hechos_consumo_agua`**
   * **Claves de Dimensión (FKs):** `id_tiempo`, `id_ubicacion`, `id_tipo_usuario`.
   * **Métricas / Medidas:** 
     * `consumo_m3`: Metros cúbicos consumidos de agua potable.
     * `monto_facturado`: Importe total facturado en pesos mexicanos (MXN).
     * `num_tomas`: Cantidad de tomas de agua asociadas al registro.

2. **Tablas de Dimensión (*Dimension Tables*):**
   * **`dim_tiempo`:** Desglosa la temporalidad del registro (`id_tiempo`, `anio`, `bimestre`, `mes`).
   * **`dim_ubicacion`:** Contiene la información geográfica (`id_ubicacion`, `alcaldia`, `colonia`, `codigo_postal`).
   * **`dim_tipo_usuario`:** Clasifica el tipo de consumo (`id_tipo_usuario`, `tipo_tarifa` [Doméstico, No Doméstico, Mixto], `indice_desarrollo`).

3. **Granularidad del Almacén:**
   * Cada renglón de la tabla de hechos representa **el consumo total acumulado (en $m^3$) y el monto cobrado para un tipo de usuario específico dentro de una colonia/alcaldía determinada en un bimestre particular**.

---

## 2. Comprobación de Claves Foráneas y Registros Huérfanos

### 2.1 Consultas de Auditoría sobre la Base de Datos Analítica

Se ejecutaron consultas con `LEFT JOIN` para determinar si existen registros en la tabla de hechos que apunten a dimensiones no catalogadas o inexistentes.

```sql
-- 1. Verificación de hechos sin dimensión de tiempo válida
SELECT h.id_hecho, h.id_tiempo
FROM hechos_consumo_agua h
LEFT JOIN dim_tiempo t ON h.id_tiempo = t.id_tiempo
WHERE t.id_tiempo IS NULL;

-- 2. Verificación de hechos sin dimensión de ubicación válida
SELECT h.id_hecho, h.id_ubicacion
FROM hechos_consumo_agua h
LEFT JOIN dim_ubicacion u ON h.id_ubicacion = u.id_ubicacion
WHERE u.id_ubicacion IS NULL;

-- 3. Verificación de hechos sin dimensión de tipo de usuario válida
SELECT h.id_hecho, h.id_tipo_usuario
FROM hechos_consumo_agua h
LEFT JOIN dim_tipo_usuario us ON h.id_tipo_usuario = us.id_tipo_usuario
WHERE us.id_tipo_usuario IS NULL;
```

**Resultado de la auditoría:** `(0 rows)` en las tres consultas. Los datos limpios del repositorio respetan la congruencia dimensional entre hechos y dimensiones.

---

## 3. Análisis: Declaración de Claves Foráneas en Almacenes de Datos (OLAP vs. OLTP)

### 3.1 Script de Modificación del Esquema (Prueba en copia de la base de datos)

En caso de que el almacén se haya construido sin restricciones explícitas de clave foránea a nivel DDL para acelerar la carga, las restricciones se agregan en un entorno de pruebas mediante el siguiente script:

```sql
-- Agregar FK de Tiempo
ALTER TABLE hechos_consumo_agua
ADD CONSTRAINT fk_hechos_tiempo
FOREIGN KEY (id_tiempo) REFERENCES dim_tiempo(id_tiempo);

-- Agregar FK de Ubicación
ALTER TABLE hechos_consumo_agua
ADD CONSTRAINT fk_hechos_ubicacion
FOREIGN KEY (id_ubicacion) REFERENCES dim_ubicacion(id_ubicacion);

-- Agregar FK de Tipo de Usuario
ALTER TABLE hechos_consumo_agua
ADD CONSTRAINT fk_hechos_tipo_usuario
FOREIGN KEY (id_tipo_usuario) REFERENCES dim_tipo_usuario(id_tipo_usuario);
```

### 3.2 Justificación: ¿Por qué un Almacén de Datos omite explícitamente las Claves Foráneas?

1. **Rendimiento de Ingesta Masiva (Carga ETL / ELT):**
   Durante los procesos de carga de millones de filas sobre datos abiertos del consumo de agua (como los datasets de SACMEX), validar las claves foráneas por cada renglón insertado alienta severamente los comandos de carga masiva (`COPY` o `LOAD DATA`).

2. **Cargas Asíncronas o Desincronizadas:**
   En entornos reales de Big Data, la facturación de consumo de agua puede procesarse antes de que una nueva colonia o reestructuración postal haya sido dada de alta en la dimensión geográfica `dim_ubicacion`.

3. **Arquitecturas Columnares / Motores OLAP:**
   Sistemas analíticos como ClickHouse, BigQuery o DuckDB prescinden de constraints rígidos de FK para habilitar paralelismo masivo y lectura columnar ultrarrápida.

### 3.3 Riesgos de Omitir las Claves Foráneas

* **Pérdida Silenciosa de Datos en Consultas (`INNER JOIN`):** Si existen hechos huérfanos y los tableros de visualización realizan joins exactos entre hechos y dimensiones, los metros cúbicos o montos de esos registros se descartarán, generando subestimaciones en el volumen total de agua consumida en la CDMX.
* **Inconsistencia en Herramientas de BI:** Los filtros de búsqueda por alcaldía o tarifa no desplegarán consumos cuyas claves no concuerden con los catálogos dimensionales.