# Ejercicio 2.4: Edición de Valores de una Base de Datos Relacional Empleando DML y Transacciones

En este documento se presentan los ejercicios de manipulación de datos (DML) y control de transacciones aplicados sobre el proyecto propio, modelando situaciones reales del negocio.

---

## 1. Inserciones Avanzadas

### 1.1 Inserción a partir de una consulta (`INSERT INTO ... SELECT`)

* **Escenario del negocio:** Registrar de forma automatizada una lista de productos en promoción dentro de una tabla de auditoría/descuentos basada en aquellos productos que tengan existencias mayores a 50 unidades.

```sql
-- Creación de la estructura de destino si no existe
CREATE TABLE IF NOT EXISTS Promociones (
    Cod_Barra VARCHAR(50) PRIMARY KEY,
    Nombre VARCHAR(100) NOT NULL,
    PrecioOriginal DECIMAL(10,2) NOT NULL,
    PrecioPromocion DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (Cod_Barra) REFERENCES Productos(Cod_Barra) ON DELETE CASCADE
);

-- Inserción basada en consulta con agregación/filtrado
INSERT INTO Promociones (Cod_Barra, Nombre, PrecioOriginal, PrecioPromocion)
SELECT 
    p.Cod_Barra,
    p.Nombre,
    p.Precio,
    ROUND(p.Precio * 0.85, 2) AS PrecioPromocion -- 15% de descuento
FROM Productos p
JOIN Inventario i ON p.Cod_Barra = i.Cod_Barra
WHERE i.Cantidad > 50;
```

### 1.2 Inserción resolviendo duplicados (UPSERT / `ON CONFLICT`)

* **Escenario del negocio:** Cargar o actualizar el catálogo de productos. Si el código de barras ya existe, se actualiza el precio de venta; si no existe, se inserta el nuevo renglón.

```sql
INSERT INTO Productos (Cod_Barra, Nombre, Marca, Precio)
VALUES ('7501000123456', 'Leche Entera 1L', 'Lala', 28.50)
ON CONFLICT (Cod_Barra) 
DO UPDATE SET 
    Precio = EXCLUDED.Precio,
    Nombre = EXCLUDED.Nombre;
```

---

## 2. Actualización de Valores (UPDATE)

### 2.1 UPDATE dependiente de otra tabla

* **Escenario del negocio:** Recalcular y actualizar el atributo `Total` de la tabla `Venta` sumando el precio real de los productos asociados en la tabla `Incluir`.

```sql
UPDATE Venta v
SET Total = (
    SELECT COALESCE(SUM(p.Precio), 0)
    FROM Incluir i
    JOIN Productos p ON i.Cod_Barra = p.Cod_Barra
    WHERE i.Ticket = v.Ticket
)
WHERE v.Ticket IN (SELECT DISTINCT Ticket FROM Incluir);
```

### 2.2 UPDATE condicional con estructura `CASE`

* **Escenario del negocio:** Clasificación o etiquetado del puesto de empleados para ajuste salarial según el total de ventas que hayan realizado en el sistema.

```sql
UPDATE Empleado
SET Puesto = CASE 
    WHEN (SELECT COUNT(*) FROM Venta WHERE Identificador_empleado = Empleado.Identificador) >= 20 THEN 'Vendedor Senior'
    WHEN (SELECT COUNT(*) FROM Venta WHERE Identificador_empleado = Empleado.Identificador) BETWEEN 10 AND 19 THEN 'Vendedor Intermedio'
    ELSE 'Vendedor Junior'
END;
```

---

## 3. Borrado Físico vs. Borrado Lógico

### 3.1 DELETE con subconsulta (Borrado Físico)

* **Escenario del negocio:** Limpiar de la base de datos a aquellos clientes que no hayan realizado ninguna compra registradas en la tabla `Venta`.

```sql
DELETE FROM Cliente
WHERE Identificador IN (
    SELECT c.Identificador
    FROM Cliente c
    LEFT JOIN Venta v ON c.Identificador = v.Identificador_cliente
    WHERE v.Ticket IS NULL
);
```

### 3.2 Borrado Lógico

* **Escenario del negocio:** Desactivar un producto del catálogo para que no se pueda seguir vendiendo, sin perder el historial contable de ventas pasadas.

```sql
-- Adición de columna para borrado lógico (si no existe)
ALTER TABLE Productos ADD COLUMN IF NOT EXISTS Activo BOOLEAN DEFAULT TRUE;

-- Ejecución del borrado lógico
UPDATE Productos
SET Activo = FALSE
WHERE Cod_Barra = '7501000123456';
```

### 3.3 Comparación: Borrado Físico vs. Borrado Lógico

| Aspecto | Borrado Físico (`DELETE`) | Borrado Lógico (`UPDATE Activo = FALSE`) |
| :--- | :--- | :--- |
| **Recuperación de datos** | Imposible a menos que se restaure un respaldo. | Inmediata (basta con cambiar el estado a `TRUE`). |
| **Integridad Referencial** | Puede chocar con `ON DELETE RESTRICT` si la clave está en uso en ventas pasadas. | No afecta las restricciones de clave foránea porque el renglón sigue existiendo. |
| **Auditoría e Históricos** | Se pierde el historial fiscal y de operaciones acumuladas. | Conserva el 100% del historial de transacciones para análisis contables. |
| **Cuándo conviene** | Datos basura, registros temporales o cancelaciones inmediatas sin consecuencias. | Entidades principales del negocio (Productos, Clientes, Empleados). |

---

## 4. Control de Transacciones Complejas

### 4.1 Transacción con Puntos de Recuperación (`SAVEPOINT`) y Reversión Parcial

* **Escenario del negocio:** Procesar una venta compuesta: registrar el ticket de venta, añadir un producto válido, intentar añadir un producto invalido/agotado (rollback al savepoint) y finalmente confirmar la venta del producto correcto.

```sql
BEGIN TRANSACTION;

-- Paso 1: Insertar la cabecera del ticket de venta
INSERT INTO Venta (Ticket, Identificador_cliente, Identificador_empleado, Total, Dia, Mes, Año, Empleadoventas)
VALUES ('TCK-2026-001', 'CLI-0001', 'EMP-0001', 150.00, 9, 11, 2026, 'Carlos Pérez');

-- Paso 2: Crear punto de recuperación tras registrar cabecera
SAVEPOINT cabecera_registrada;

-- Paso 3: Insertar primer producto en la venta
INSERT INTO Incluir (Ticket, Cod_Barra)
VALUES ('TCK-2026-001', '7501000123456');

-- Paso 4: Crear punto de recuperación previo a inserción dudosa
SAVEPOINT primer_producto;

-- Paso 5: Intento de insertar un producto con código errado (simulación)
INSERT INTO Incluir (Ticket, Cod_Barra)
VALUES ('TCK-2026-001', 'CODIGO-INCORRECTO');

-- Paso 6: Revertir únicamente el segundo producto al fallar la validación de negocio
ROLLBACK TO SAVEPOINT primer_producto;

-- Paso 7: Confirmar la transacción con la cabecera y el producto válido
COMMIT;
```

---

## 5. Transacción con Fallo por Violación de Restricción (Atomicidad)

### 5.1 Ejecución del script transaccional con error provocado

* **Escenario del negocio:** Se intenta registrar una venta con un precio total negativo (violación de restricción `CHECK (Total >= 0)`).

```sql
BEGIN TRANSACTION;

-- Operación 1: Correcta
INSERT INTO Venta (Ticket, Identificador_cliente, Identificador_empleado, Total, Dia, Mes, Año, Empleadoventas)
VALUES ('TCK-FAIL-01', 'CLI-0001', 'EMP-0001', 100.00, 9, 11, 2026, 'Carlos Pérez');

-- Operación 2: Inserción violando CHECK (Total negativo)
INSERT INTO Venta (Ticket, Identificador_cliente, Identificador_empleado, Total, Dia, Mes, Año, Empleadoventas)
VALUES ('TCK-FAIL-02', 'CLI-0001', 'EMP-0001', -50.00, 9, 11, 2026, 'Carlos Pérez');

COMMIT;
```

### 5.2 Respuesta y mensaje de error del gestor (PostgreSQL)

```
ERROR: new row for relation "venta" violates check constraint "venta_total_check"
DETAIL: Failing row contains (TCK-FAIL-02, CLI-0001, EMP-0001, -50.00, 9, 11, 2026, Carlos Pérez).
SQL state: 23514
```

### 5.3 Comprobación de que la base de datos quedó en su estado original

```sql
SELECT COUNT(*) FROM Venta WHERE Ticket IN ('TCK-FAIL-01', 'TCK-FAIL-02');
-- Resultado: 0 (Demuestra que la Operación 1 fue revertida automáticamente al abortar la transacción)
```