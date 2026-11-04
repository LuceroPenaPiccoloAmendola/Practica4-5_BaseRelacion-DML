# Ejercicio 1.1: Tipos de datos y dominios del sistema de supermercado

En esta sección se documenta el diseño de los tipos de datos y los dominios personalizados implementados en PostgreSQL para el sistema del supermercado. El uso de dominios reutilizables (CREATE DOMAIN) permite estandarizar validaciones y asegurar la integridad a nivel de esquema en toda la base de datos.

## 1. Dominios personalizados (CREATE DOMAIN)

```sql
-- Dominio para montos monetarios y precios (evita números en punto flotante y asegura valores positivos)
CREATE DOMAIN dm_monto AS NUMERIC(10, 2)
    CHECK (VALUE >= 0.00);

-- Dominio para correos electrónicos con validación de formato mediante expresión regular
CREATE DOMAIN dm_email AS VARCHAR(150)
    CHECK (VALUE ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');

-- Dominio para números telefónicos de longitud fija (10 dígitos)
CREATE DOMAIN dm_telefono AS VARCHAR(10)
    CHECK (VALUE ~ '^[0-9]{10}$');
```

## 2. Tabla de tipo de datos y dominio del sistema

| Tabla | Columna | Tipo de Dato | Longitud | Justificación |
| :--- | :--- | :--- | :--- | :--- |
| **Persona** | `Identificador` | `VARCHAR` | 20 | Permite almacenar claves alfanuméricas únicas de identificación (como CURP, RFC o ID interno). |
| | `PrimerNombre` | `VARCHAR` | 50 | Espacio suficiente para almacenar nombres compuestos o estándar. |
| | `SegundoNombre` | `VARCHAR` | 50 | Opcional para registrar el segundo nombre de la persona. |
| | `ApellidoPaterno` | `VARCHAR` | 50 | Espacio adecuado para el apellido paterno. |
| | `ApellidoMaterno` | `VARCHAR` | 50 | Espacio adecuado para el apellido materno. |
| | `TipoPersona` | `VARCHAR` | 20 | Clasifica el rol de la persona dentro del sistema (Cliente, Empleado, Proveedor). |
| **Cliente** | `Identificador` | `VARCHAR` | 20 | Llave primaria y foránea que referencia a `Persona`. |
| **Empleado** | `Identificador` | `VARCHAR` | 20 | Llave primaria y foránea que referencia a `Persona`. |
| | `Puesto` | `VARCHAR` | 50 | Define el cargo o rol laboral del empleado en el supermercado. |
| **Proveedores** | `Identificador` | `VARCHAR` | 20 | Llave primaria y foránea que referencia a `Persona`. |
| | `Calle` | `VARCHAR` | 100 | Longitud estándar para nombres de calles o avenidas. |
| | `Ciudad` | `VARCHAR` | 50 | Suficiente para almacenar la ciudad o municipio del proveedor. |
| | `Num_Calle` | `VARCHAR` | 10 | Permite números exteriores e interiores (ej. "123-A", "S/N"). |
| **PersonaTeléfono** | `Identificador` | `VARCHAR` | 20 | Relaciona el teléfono con la persona propietaria. |
| | `Teléfono` | `VARCHAR` | 15 | Almacena números locales o internacionales con flexibilidad de longitud. |
| **ClienteCorreo** | `Identificador` | `VARCHAR` | 20 | Identifica al cliente asociado al correo electrónico. |
| | `Correo` | `VARCHAR` | 100 | Longitud estándar para direcciones de correo electrónico. |
| **ProveedorCorreo** | `Identificador` | `VARCHAR` | 20 | Identifica al proveedor asociado al correo electrónico. |
| | `Correo` | `VARCHAR` | 100 | Longitud estándar para direcciones de correo electrónico. |
| **Productos** | `Cod_Barra` | `VARCHAR` | 50 | Soporta códigos de barras estándares (EAN-13, UPC, códigos internos alfanuméricos). |
| | `Nombre` | `VARCHAR` | 100 | Nombre descriptivo del producto en aparador. |
| | `Marca` | `VARCHAR` | 50 | Marca comercial del producto. |
| | `Precio` | `DECIMAL` | (10, 2) | Precisión monetaria exacta (evita errores de punto flotante). Restricción `>= 0`. |
| **Categoria** | `Cod_Barra` | `VARCHAR` | 50 | Llave foránea vinculada al producto clasificado. |
| | `Identificador` | `VARCHAR` | 50 | Código o identificador único de la categoría. |
| | `Nombre` | `VARCHAR` | 50 | Nombre descriptivo de la categoría (ej. Lácteos, Electrónica). |
| **Inventario** | `Cod_Barra` | `VARCHAR` | 50 | Llave foránea que referencia al producto en inventario. |
| | `Fecha_Actu` | `DATE` | - | Fecha de actualización o registro del stock. |
| | `Cantidad` | `INT` | - | Cantidad física disponible. Restricción `>= 0`. |
| **Surtir** | `Cod_Barra` | `VARCHAR` | 50 | Producto que es abastecido por el proveedor. |
| | `Identificador` | `VARCHAR` | 20 | Proveedor encargado de surtir el producto. |
| **Venta** | `Ticket` | `VARCHAR` | 20 | Identificador único del comprobante o ticket de venta. |
| | `Identificador_cliente` | `VARCHAR` | 20 | Cliente que realizó la compra (Llave foránea). |
| | `Identificador_empleado` | `VARCHAR` | 20 | Empleado que atendió la transacción (Llave foránea). |
| | `Total` | `DECIMAL` | (10, 2) | Monto total de la venta con precisión monetaria. Restricción `>= 0`. |
| | `Dia` | `INT` | - | Día en que se efectúa la venta. |
| | `Mes` | `INT` | - | Mes en que se efectúa la venta. |
| | `Año` | `INT` | - | Año en que se efectúa la venta. |
| | `Empleadoventas` | `VARCHAR` | 50 | Descripción adicional o nombre del cajero/vendedor en turno. |
| **Incluir** | `Ticket` | `VARCHAR` | 20 | Referencia a la venta en la que se incluye el producto. |
| | `Cod_Barra` | `VARCHAR` | 50 | Producto incluido en el detalle de la venta. |

---

# Ejercicio 1.2: Restricciones de Columna y Reglas del Dominio

En este ejercicio se declaran las restricciones de integridad y validación aplicadas a las tablas del sistema de supermercado, garantizando que cada regla de negocio se cumpla directamente a nivel de base de datos.

## 1. Declaración de Restricciones y Reglas de Negocio

1. **NOT NULL en atributos esenciales (Persona.PrimerNombre, Persona.ApellidoPaterno):**
   * *Regla de negocio:* Ninguna persona (cliente, empleado o proveedor) puede registrarse en el sistema sin su primer nombre y apellido paterno.
2. **NOT NULL en claves foráneas obligatorias (Venta.Identificador_cliente, Venta.Identificador_empleado):**
   * *Regla de negocio:* Toda venta registrada en caja debe estar asociada obligatoriamente a un cliente y al empleado que procesó la transacción.
3. **NOT NULL en identificadores clave (Productos.Nombre, Categoria.Nombre):**
   * *Regla de negocio:* Es obligatorio asignarle un nombre descriptivo a cualquier producto registrado en catálogo y a las categorías existentes.
4. **UNIQUE simple (PersonaTeléfono.Teléfono):**
   * *Regla de negocio:* Un número telefónico no puede ser asignado a múltiples registros duplicados para evitar conflictos de contacto.
5. **UNIQUE compuesta (ClienteCorreo.Identificador, ClienteCorreo.Correo):**
   * *Regla de negocio:* Un cliente no puede tener registrado el mismo correo electrónico más de una vez en la tabla multivaluada.
6. **CHECK de valor no negativo (Productos.Precio >= 0):**
   * *Regla de negocio:* El precio de venta al público de un producto jamás puede ser un valor negativo.
7. **CHECK de stock válido (Inventario.Cantidad >= 0):**
   * *Regla de negocio:* La cantidad física de existencias en el inventario no puede registrar valores bajo cero.
8. **CHECK de total de venta (Venta.Total >= 0):**
   * *Regla de negocio:* El importe total calculado en un ticket de venta debe ser igual o superior a cero.
9. **CHECK de rango de fechas / calendario (Venta.Dia, Venta.Mes, Venta.Año):**
   * *Regla de negocio:* Los campos de fecha en la tabla venta deben representar valores válidos de calendario (Dia entre 1 y 31, Mes entre 1 y 12, Año mayor a 2000).
10. **CHECK de formato de texto o categorías permitidas (Persona.TipoPersona):**
    * *Regla de negocio:* El tipo de persona registrado en el sistema únicamente puede tomar los valores definidos: `'Cliente'`, `'Empleado'` o `'Proveedor'`.

---

## 2. Script de Migración SQL (001-dominios.sql)

```sql

ALTER TABLE Persona
    ADD CONSTRAINT chk_persona_tipo 
    CHECK (TipoPersona IN ('Cliente', 'Empleado', 'Proveedor'));

ALTER TABLE Venta
    ADD CONSTRAINT chk_venta_dia CHECK (Dia BETWEEN 1 AND 31),
    ADD CONSTRAINT chk_venta_mes CHECK (Mes BETWEEN 1 AND 12),
    ADD CONSTRAINT chk_venta_anio CHECK (Año >= 2024);

ALTER TABLE PersonaTeléfono
    ADD CONSTRAINT unq_persona_telefono UNIQUE (Teléfono);

ALTER TABLE ClienteCorreo
    ADD CONSTRAINT chk_cliente_correo_formato 
    CHECK (Correo ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');

ALTER TABLE ProveedorCorreo
    ADD CONSTRAINT chk_proveedor_correo_formato 
    CHECK (Correo ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');
```

<img width="1366" height="730" alt="E1JPG" src="https://github.com/user-attachments/assets/9c43e91f-6e0e-4a0c-93a6-ab32ceed7ed6" />
