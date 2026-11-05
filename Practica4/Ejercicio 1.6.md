# Ejercicio 1.6: Gestión de usuarios y permisos

En esta sección se documenta la configuración de seguridad de la base de datos del supermercado, implementando roles especializados bajo el principio de mínimo privilegio para evitar el uso continuo del usuario administrador y asegurar que cada componente del sistema acceda únicamente a lo estrictamente necesario.

---

## 1. Definición de roles del sistema

Se establecieron tres roles principales con propósitos específicos:

1. **Rol de Aplicación (`rol_supermercado_app`):** 
   * *Propósito:* Utilizado exclusivamente por el backend o el script de carga automatizada para la operación diaria. Tiene permisos de lectura y escritura (`SELECT`, `INSERT`, `UPDATE`, `DELETE`) únicamente en las tablas operativas y transaccionales que requiere el sistema.
2. **Rol de Consulta (`rol_supermercado_lector`):** 
   * *Propósito:* Diseñado para tareas de auditoría, reportes o consultas analíticas. Posee permisos estrictamente de solo lectura (`SELECT`) sobre los catálogos e inventarios, sin facultades para alterar o eliminar datos.
3. **Rol de Administración (`rol_supermercado_admin`):** 
   * *Propósito:* Rol con privilegios elevados para la gestión de esquemas, migraciones y mantenimiento general del SGBD.

---

## 2. Sentencias DDL para creación de roles y asignación de permisos

Los usuarios y roles se configuran utilizando variables de entorno para las contraseñas, asegurando que no queden credenciales hardcodeadas en el repositorio:

```sql
-- Creación de roles base
CREATE ROLE rol_supermercado_app LOGIN PASSWORD '${APP_PASSWORD}';
CREATE ROLE rol_supermercado_lector LOGIN PASSWORD '${READER_PASSWORD}';
CREATE ROLE rol_supermercado_admin LOGIN PASSWORD '${ADMIN_PASSWORD}';

-- Otorgar privilegios mínimos al rol de aplicación
GRANT CONNECT ON DATABASE db_supermercado TO rol_supermercado_app;
GRANT USAGE ON SCHEMA public TO rol_supermercado_app;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO rol_supermercado_app;

-- Otorgar privilegios de solo lectura al rol de consulta
GRANT CONNECT ON DATABASE db_supermercado TO rol_supermercado_lector;
GRANT USAGE ON SCHEMA public TO rol_supermercado_lector;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO rol_supermercado_lector;

```

---

## 3. Evidencias

<img width="1366" height="694" alt="E2" src="https://github.com/user-attachments/assets/04cae501-1cc8-4e0b-ad11-b6e23c06c899" />

<img width="1366" height="570" alt="E4" src="https://github.com/user-attachments/assets/bf7dccbf-f290-494a-bd5f-4fc016a74558" />

<img width="1366" height="550" alt="E3" src="https://github.com/user-attachments/assets/b7c2f707-d424-40b1-8ca1-da16e6350cad" />

<img width="1366" height="522" alt="E5" src="https://github.com/user-attachments/assets/c8135b26-f6d2-44c9-bb72-157098d581d8" />


