# Ejercicio 2.1: Las claves foráneas del proyecto propio

En el presente documento se detallan las claves foráneas (FK) implementadas en el esquema relacional del proyecto propio. Cada entrada indica la relación correspondiente en el modelo EER, las tablas y columnas involucradas, la admisible nulabilidad conforme a la cardinalidad mínima del modelo conceptual y la justificación del dominio.

## Tabla de Claves Foráneas

| Tabla Local | Columna(s) Local(es) | Tabla Referenciada (Padre) | Columna Referenciada | Relación en el Modelo EER | ¿Admite Nulos? | Justificación de Nulabilidad |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
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
