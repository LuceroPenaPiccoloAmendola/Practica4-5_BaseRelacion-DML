# Ejercicio 2.3: Pruebas de Integridad Referencial

En este documento se presentan las pruebas ejecutadas sobre la base de datos relacional para validar el comportamiento de la integridad referencial, el cumplimiento de las acciones de borrado y actualización, así como la detección de inconsistencias.

---

## 1. Pruebas de Acciones Referenciales con Datos Reales

### Prueba 1.1: `ON DELETE CASCADE` (Borrado en cascada en Herencia y Atributos Multivaluados)

* **Objetivo:** Verificar que al borrar una tupla de la tabla padre (`Persona`), sus tuplas asociadas en la subentidad (`Cliente`) y atributos multivaluados (`PersonaTeléfono`) se eliminen automáticamente.

#### Estado inicial (Conteos previos)
```sql
SELECT COUNT(*) FROM Persona WHERE Identificador = 'PER-0099';
-- Resultado: 1

SELECT COUNT(*) FROM Cliente WHERE Identificador = 'PER-0099';
-- Resultado: 1

SELECT COUNT(*) FROM PersonaTeléfono WHERE Identificador = 'PER-0099';
-- Resultado: 2
```

#### Sentencia DML ejecutada
```sql
DELETE FROM Persona WHERE Identificador = 'PER-0099';
```

#### Estado final (Conteos posteriores)
```sql
SELECT COUNT(*) FROM Persona WHERE Identificador = 'PER-0099';
-- Resultado: 0

SELECT COUNT(*) FROM Cliente WHERE Identificador = 'PER-0099';
-- Resultado: 0 (Eliminado en cascada)

SELECT COUNT(*) FROM PersonaTeléfono WHERE Identificador = 'PER-0099';
-- Resultado: 0 (Eliminado en cascada)
```

---

### Prueba 1.2: `ON DELETE RESTRICT` (Restricción de borrado para proteger historial contable)

* **Objetivo:** Comprobar que el gestor impida la eliminación de un empleado si este ya ha registrado ventas en el sistema.

#### Estado inicial
```sql
SELECT Identificador, Puesto FROM Empleado WHERE Identificador = 'EMP-0001';
-- Resultado: 1 renglón (EMP-0001, Cajero)

SELECT COUNT(*) FROM Venta WHERE Identificador_empleado = 'EMP-0001';
-- Resultado: 15 ventas registradas
```

#### Sentencia DML ejecutada
```sql
DELETE FROM Empleado WHERE Identificador = 'EMP-0001';
```

#### Mensaje devuelto por PostgreSQL
```text
ERROR: update or delete on table "empleado" violates foreign key constraint "venta_identificador_empleado_fkey" on table "venta"
DETAIL: Key (identificador)=(EMP-0001) is still referenced from table "venta".
SQL state: 23503
```

#### Estado final
```sql
SELECT COUNT(*) FROM Empleado WHERE Identificador = 'EMP-0001';
-- Resultado: 1 (El registro fue protegido correctamente y NO se borró)
```

---

## 2. Intento de Inserción con Clave Foránea Inexistente

* **Objetivo:** Demostrar el rechazo de inserción en la tabla `Venta` referenciando a un cliente que no existe en la tabla `Cliente`.

#### Sentencia DML ejecutada
```sql
INSERT INTO Venta (Ticket, Identificador_cliente, Identificador_empleado, Total, Dia, Mes, Año, Empleadoventas)
VALUES ('TCK-99999', 'CLI-INEXISTENTE', 'EMP-0001', 450.50, 15, 10, 2026, 'Juan Pérez');
```

#### Mensaje devuelto por PostgreSQL
```text
ERROR: insert or update on table "venta" violates foreign key constraint "venta_identificador_cliente_fkey" on table "cliente"
DETAIL: Key (identificador_cliente)=(CLI-INEXISTENTE) is not present in table "cliente".
SQL state: 23503
```

---

## 3. Demostración del Orden de Carga

### 3.1 Intento de carga desordenada (Hija antes que Padre)

Si el script de automatización intenta popular la tabla `ClienteCorreo` antes de haber insertado los registros correspondientes en la tabla padre `Cliente`:

```sql
INSERT INTO ClienteCorreo (Identificador, Correo) VALUES ('CLI-0050', 'cliente50@gmail.com');
```

**Resultado/Mensaje de error:**
```text
ERROR: insert or update on table "clientecorreo" violates foreign key constraint "clientecorreo_identificador_fkey" on table "cliente"
DETAIL: Key (identificador)=(CLI-0050) is not present in table "cliente".
SQL state: 23503
```

### 3.2 Solución mediante Orden Topológico de Carga

El script de carga soluciona esto garantizando la ejecución de inserciones en el siguiente orden estricto de dependencia:

1. **Nivel 0 (Entidades base independientes):** `Persona`, `Productos`
2. **Nivel 1 (Subtipos y entidades dependientes directas):** `Cliente`, `Empleado`, `Proveedores`, `Categoria`, `Inventario`
3. **Nivel 2 (Atributos multivaluados y relaciones interactivas):** `PersonaTeléfono`, `ClienteCorreo`, `ProveedorCorreo`, `Surtir`
4. **Nivel 3 (Transacciones de cabecera):** `Venta`
5. **Nivel 4 (Detalle de transacciones / Tablas puente de nivel superior):** `Incluir`

---

## 4. Consultas de Verificación de Integridad Referencial

Las siguientes consultas de auditoría fueron diseñadas para detectar violaciones a la integridad en la base de datos. En una base limpia y consistente, **todas las consultas deben devolver 0 registros (resultado vacío)**.

### 4.1 Verificación de Claves Foráneas Huérfanas

#### Consulta A: Ventas asociadas a clientes inexistentes
```sql
SELECT v.Ticket, v.Identificador_cliente
FROM Venta v
LEFT JOIN Cliente c ON v.Identificador_cliente = c.Identificador
WHERE c.Identificador IS NULL;
```
* **Resultado:** `(0 rows)`

#### Consulta B: Detalle de inclusión de ventas con productos inexistentes
```sql
SELECT i.Ticket, i.Cod_Barra
FROM Incluir i
LEFT JOIN Productos p ON i.Cod_Barra = p.Cod_Barra
WHERE p.Cod_Barra IS NULL;
```
* **Resultado:** `(0 rows)`

### 4.2 Verificación de Duplicados en Columnas Únicas / Claves Primarias Compuestas

#### Consulta: Registros duplicados en detalles de ventas (`Incluir`)
```sql
SELECT Ticket, Cod_Barra, COUNT(*) 
FROM Incluir 
GROUP BY Ticket, Cod_Barra 
HAVING COUNT(*) > 1;
```
* **Resultado:** `(0 rows)`

### 4.3 Verificación de Nulos en Columnas No Permitidas (`NOT NULL`)

#### Consulta: Ventas sin empleado asignado o con total nulo
```sql
SELECT Ticket 
FROM Venta 
WHERE Identificador_empleado IS NULL 
   OR Identificador_cliente IS NULL 
   OR Total IS NULL;
```
* **Resultado:** `(0 rows)`