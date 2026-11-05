# Análisis de acciones referenciales en base de datos: Dominio Supermercado

**Alumno:** Fiorella Karina del Carmen Piccolo Amendola  
**Asignatura:** Bases de Datos  
**Relación seleccionada:** `Categoria` (Tabla Padre), `Producto` (Tabla Hijo)

---

## 1. Declaración de la relación y restricción

Se analiza la dependencia existencial entre las categorías de artículos y los productos registrados en el inventario del supermercado.

* **Tabla Padre:** `Categoria` (`id_categoria` PK)
* **Tabla Hija:** `Producto` (`id_categoria` FK)

---

## 2. Modificación de la acción referencial (DDL)

Se modifica la llave foránea en la base de datos de pruebas para cambiar el comportamiento original (`RESTRICT`) por **`CASCADE`**.

```sql
-- 1. Eliminar la restricción de llave foránea existente
ALTER TABLE Producto 
DROP CONSTRAINT fk_producto_categoria;

-- 2. Crear la llave foránea con la nueva acción referencial ON DELETE CASCADE
ALTER TABLE Producto 
ADD CONSTRAINT fk_producto_categoria 
FOREIGN KEY (id_categoria) 
REFERENCES Categoria(id_categoria) 
ON DELETE CASCADE;
```

---

## 3. Demostración práctica con datos

-- Consultar la categoría a eliminar (ejemplo: Categoria ID 5 = 'Lácteos')
SELECT id_categoria, nombre_categoria 
FROM Categoria 
WHERE id_categoria = 5;

-- Consultar los productos asociados a dicha categoría
SELECT id_producto, nombre_producto, id_categoria 
FROM Producto 
WHERE id_categoria = 5;

-- Se elimina la categoría padre de la base de datos
DELETE FROM Categoria 
WHERE id_categoria = 5;

-- Verificar la tabla Padre
SELECT * FROM Categoria WHERE id_categoria = 5;
-- Resultado: 0 filas devueltas.

-- Verificar la tabla Hija
SELECT * FROM Producto WHERE id_categoria = 5;
-- Resultado: 0 filas devueltas.
