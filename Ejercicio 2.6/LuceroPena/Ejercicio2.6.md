# Ejercicio 2.6: Análisis Individual de Acciones Referenciales

**Alumno:** Lucero Peña Piccolo Amendola  
**Unidad de Aprendizaje:** Bases de Datos  
**Grupo:** 2CV2  
**Proyecto Propio:** Sistema de Gestión y Puntos de Venta para Supermercado  

---

## 1. Relación Seleccionada del Proyecto Propio

* **Tabla Padre:** `Empleado`
* **Tabla Hija:** `Venta`
* **Clave Foránea:** `Venta.Identificador_empleado` $\rightarrow$ `Empleado.Identificador`
* **Acción Referencial Original en Producción:** `ON DELETE RESTRICT`

---

## 2. Definición del Experimento

El objetivo de esta prueba es evaluar el impacto en la integridad del negocio al cambiar la regla de eliminación referencial de `ON DELETE RESTRICT` a `ON DELETE CASCADE` en una copia local de la base de datos de pruebas.

### Configuración en la copia de la base de datos

```sql
-- 1. Eliminar la restricción original en la copia
ALTER TABLE Venta 
DROP CONSTRAINT IF EXISTS venta_identificador_empleado_fkey;

-- 2. Aplicar la modificación a ON DELETE CASCADE
ALTER TABLE Venta 
ADD CONSTRAINT venta_identificador_empleado_fkey 
FOREIGN KEY (Identificador_empleado) 
REFERENCES Empleado(Identificador) 
ON DELETE CASCADE 
ON UPDATE CASCADE;
```

---

## 3. Demostración con Datos Reales

### 3.1 Estado Inicial de la Base de Datos

Se toma un empleado específico (`EMP-0005`, correspondiente a un cajero) y sus registros de ventas asociadas en la tabla `Venta`:

```sql
-- Estado antes de la eliminación
SELECT Identificador, Puesto FROM Empleado WHERE Identificador = 'EMP-0005';
-- Resultado: 1 renglón (EMP-0005 | Cajero)

SELECT COUNT(*) AS VentasRegistradas FROM Venta WHERE Identificador_empleado = 'EMP-0005';
-- Resultado: 42 ventas

SELECT SUM(Total) AS IngresosGenerados FROM Venta WHERE Identificador_empleado = 'EMP-0005';
-- Resultado: $18,450.00 MXN
```

### 3.2 Execución del Borrado del Empleado (`DELETE`)

```sql
DELETE FROM Empleado WHERE Identificador = 'EMP-0005';
-- Comentario del gestor: DELETE 1 (La eliminación se ejecutó sin errores)
```

### 3.3 Estado Posterior en la Base de Datos

```sql
-- Verificar tabla Empleado
SELECT COUNT(*) FROM Empleado WHERE Identificador = 'EMP-0005';
-- Resultado: 0

-- Verificar tabla Venta
SELECT COUNT(*) FROM Venta WHERE Identificador_empleado = 'EMP-0005';
-- Resultado: 0 (Se borraron las 42 ventas automáticamente)
```

---

## 4. Análsis Comparativo: ¿Qué se pierde vs. Qué se conserva?

| Aspecto Analizado | Con `ON DELETE RESTRICT` (Diseño Original) | Con `ON DELETE CASCADE` (Diseño Alterado) |
| :--- | :--- | :--- |
| **Integridad Contable/Fiscal** | **Conservada:** No permite eliminar al empleado mientras existan comprobantes de venta registrados bajo su nombre. | **Destruida:** La base de datos elimina automáticamente los $18,450.00 MXN en registros de ventas. |
| **Auditoría Interna** | **Garantizada:** Se mantiene la trazabilidad de qué cajero atendió cada compra. | **Perdida:** Se generan descuadres de caja e imprecisiones en los cierres de inventario y corte contable. |
| **Mantenimiento de Catálogos** | **Exigente:** Para dar de baja a un empleado de la plantilla activa, se requiere implementar **borrado lógico** (`UPDATE Empleado SET Activo = FALSE`). | **Peligroso:** Permitir la eliminación física borra en cascada la historia operativa del supermercado. |

---

## 5. Conclusión y Decisión del Dominio

Para el dominio de un **Supermercado / Punto de Venta**, la acción referencial correcta es **`ON DELETE RESTRICT`**. 

Permitir `ON DELETE CASCADE` en la relación `Empleado` $\rightarrow$ `Venta` violaría la regla fundamental del negocio sobre **inmutabilidad de los registros contables y trazabilidad de caja**. Si un empleado renuncia o es despedido, su registro no debe eliminarse físicamente de la base de datos, sino que debe marcarse como inactivo a través de un borrado lógico, preservando la integridad de todas las ventas históricas que generó.