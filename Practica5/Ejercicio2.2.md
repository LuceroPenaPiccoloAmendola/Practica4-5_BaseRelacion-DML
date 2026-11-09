# Ejercicio 2.1 y 2.2: Claves Foráneas y Acciones Referenciales del Proyecto Propio

En el presente documento se detallan las claves foráneas (FK) implementadas en el esquema relacional del proyecto propio, sus nulabilidades y el análisis detallado de las acciones referenciales (`ON DELETE` y `ON UPDATE`) justificadas mediante reglas del dominio.

---

## Ejercicio 2.1: Tabla de Claves Foráneas

| Tabla Local | Columna(s) Local(es) | Tabla Referenciada (Padre) | Columna Referenciada | Relación en el Modelo EER | ¿Admite Nulos? | Justificación de Nulabilidad |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| **Cliente** | `Identificador` | `Persona` | `Identificador` | Especialización / Herencia | No (`NOT NULL`) | Participación total: todo cliente debe ser primeramente una persona registrada. |
| **Empleado** | `Identificador` | `Persona` | `Identificador` | Especialización / Herencia | No (`NOT NULL`) | Participación total: todo empleado debe ser primeramente una persona registrada. |
| **Proveedores** | `Identificador` | `Persona` | `Identificador` | Especialización / Herencia | No (`NOT NULL`) | Participación total: todo proveedor debe ser primeramente una persona registrada. |
| **PersonaTeléfono** | `Identificador` | `Persona` | `Identificador` | Atributo multivaluado | No (`NOT NULL`) | El teléfono no tiene sentido de existencia sin la persona a la cual pertenece. Forma parte de la PK compuesta. |
| **ClienteCorreo** | `Identificador` | `Cliente` | `Identificador` | Atributo multivaluado | No (`NOT NULL`) | El correo electrónico pertenece exclusivamente a un cliente registrado y forma parte de la PK compuesta. |
| **ProveedorCorreo** | `Identificador` | `Proveedores` | `Identificador` | Atributo multivaluado | No (`NOT NULL`) | El correo electrónico pertenece exclusivamente a un proveedor registrado y forma parte de la PK compuesta. |
| **Categoria** | `Cod_Barra` | `Productos` | `Cod_Barra` | Entidad débil / Clasificación | No (`NOT NULL`) | La categoría depende de la existencia previa del producto al que clasifica. |
| **Inventario** | `Cod_Barra` | `Productos` | `Cod_Barra` | Entidad débil / Historial de Stock | No (`NOT NULL`) | Un registro de actualización de inventario debe asociarse obligatoriamente a un producto del catálogo. |
| **Surtir** | `Cod_Barra` | `Productos` | `Cod_Barra` | Relación N:M (Productos - Proveedores) | No (`NOT NULL`) | Es la clave foránea parcial de la tabla puente; requiere un producto obligatorio. |
| **Surtir** | `Identificador` | `Proveedores` | `Identificador` | Relación N:M (Productos - Proveedores) | No (`NOT NULL`) | Es la clave foránea parcial de la tabla puente; requiere un proveedor obligatorio. |
| **Venta** | `Identificador_cliente` | `Cliente` | `Identificador` | Relación 1:N (Cliente - Venta) | No (`NOT NULL`) | Toda nota de venta debe registrar a qué cliente se le realizó la transacción. |
| **Venta** | `Identificador_empleado` | `Empleado` | `Identificador` | Relación 1:N (Empleado - Venta) | No (`NOT NULL`) | Toda venta debe registrar obligatoriamente al empleado que atendió la transacción. |
| **Incluir** | `Ticket` | `Venta` | `Ticket` | Relación N:M (Venta - Productos) | No (`NOT NULL`) | Representa el detalle del ticket de venta; depende de la cabecera del ticket. |
| **Incluir** | `Cod_Barra` | `Productos` | `Cod_Barra` | Relación N:M (Venta - Productos) | No (`NOT NULL`) | Representa los artículos incluidos en el ticket; depende de la existencia del producto. |

---

## Ejercicio 2.2: Matriz de Acciones Referenciales y Reglas de Negocio

A continuación se fundamenta cada una de las opciones de eliminación e inserción/modificación elegidas para asegurar la integridad referencial en la base de datos.

| Tabla Origen (Hijo) | Clave Foránea | Tabla Destino (Padre) | Acción `ON DELETE` | Acción `ON UPDATE` | Regla de Negocio / Justificación del Dominio |
| ----- | ----- | ----- | ----- | ----- | ----- |
| **Cliente** | `Identificador` | `Persona` | `CASCADE` | `CASCADE` | Si se elimina el registro base de una persona, el perfil de cliente pierde su entidad y debe borrarse en cascada. Si cambia la identificación de la persona, debe propagarse. |
| **Empleado** | `Identificador` | `Persona` | `CASCADE` | `CASCADE` | Si se elimina el registro base de la persona, el perfil de empleado debe borrarse automáticamente. |
| **Proveedores** | `Identificador` | `Persona` | `CASCADE` | `CASCADE` | Al eliminar la entidad persona, la subentidad proveedor debe eliminarse en cascada. |
| **PersonaTeléfono**| `Identificador` | `Persona` | `CASCADE` | `CASCADE` | Los teléfonos son atributos multivaluados; si se borra la persona, sus números telefónicos asociados no deben quedar huérfanos. |
| **ClienteCorreo** | `Identificador` | `Cliente` | `CASCADE` | `CASCADE` | Atributo multivaluado; si se remueve el cliente, sus correos asociados se eliminan de forma directa. |
| **ProveedorCorreo**| `Identificador` | `Proveedores` | `CASCADE` | `CASCADE` | Atributo multivaluado del proveedor; se remueve en cascada si se elimina la empresa/proveedor. |
| **Categoria** | `Cod_Barra` | `Productos` | `CASCADE` | `CASCADE` | La relación de categorización depende totalmente de la existencia del producto. |
| **Inventario** | `Cod_Barra` | `Productos` | `CASCADE` | `CASCADE` | El historial de existencias de un producto pierde su significado de negocio si el producto es eliminado físicamente del catálogo. |
| **Surtir** | `Cod_Barra` | `Productos` | `CASCADE` | `CASCADE` | Si se borra un producto del catálogo, la relación de reabastecimiento con el proveedor se elimina en cascada. |
| **Surtir** | `Identificador` | `Proveedores` | `CASCADE` | `CASCADE` | Si un proveedor es retirado, los registros de abastecimiento asociados en la tabla intermedia se borran. |
| **Venta** | `Identificador_cliente` | `Cliente` | **`RESTRICT`** | `CASCADE` | **Protección contable:** No se permite eliminar un cliente que tenga ventas históricas registradas en la empresa. |
| **Venta** | `Identificador_empleado` | `Empleado` | **`RESTRICT`** | `CASCADE` | **Auditoría:** No se permite eliminar la ficha de un empleado si existen tickets de venta generados bajo su autoría. |
| **Incluir** | `Ticket` | `Venta` | `CASCADE` | `CASCADE` | Si se cancela o elimina un ticket de venta (cabecera), sus renglones de detalle en la tabla `Incluir` deben eliminarse en cascada. |
| **Incluir** | `Cod_Barra` | `Productos` | **`RESTRICT`** | `CASCADE` | **Histórico comercial:** No se puede borrar un producto si este ya fue vendido en al menos un ticket registrado en la base de datos. |

---

### Resumen de Decisiones de Diseño

1. **`ON DELETE CASCADE`:** Se utiliza exclusivamente en relaciones de herencia (subtipos), atributos multivaluados y tablas puente N:M.
2. **`ON DELETE RESTRICT`:** Se impone rigurosamente en las tablas transaccionales y contables (`Venta` e `Incluir`). Esto garantiza que no se rompan las declaraciones de ingresos ni el historial fiscal o de auditoría eliminando clientes, empleados o productos en uso.
3. **`ON UPDATE CASCADE`:** Se aplica globalmente para permitir actualizaciones seguras en las claves primarias sin causar fallos por violación de integridad.