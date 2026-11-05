# Ejercicio 1.4: Automatización de la carga masiva de datos

## 1. Descripción del proceso 
Esta parte documenta el desarrollo y la ejecución del script de carga automatizada (`carga_datos.sh`), el cual constituyó uno de los retos del proyecto. Para cumplir con la regla de que la carga de datos no debe realizarse de forma manual y debe poder repetirse con un solo comando, se diseñó una solución integral en un entorno de shell script y PostgreSQL.

Entre los principales retos se encuentran:
* **Respeto a la integridad referencial:** Debido a las estrictas dependencias del modelo relacional del supermercado, las tablas maestras (como `Persona`, `Clientes`, `Empleados` y `Proveedores`) debían cargarse de forma estrictamente secuencial antes que las tablas transaccionales y de detalle (como `Inventario`, `Venta` e `Incluir`).
* **Manejo de errores y limpieza previa:** Se configuró el script para limpiar la base de datos de forma segura (`TRUNCATE` o reinicio de esquemas) antes de cada ejecución, evitando conflictos de llaves duplicadas si el script se ejecutaba múltiples veces.
* **Automatización con un solo comando:** Se integró la lectura automatizada de todos los archivos `.csv` generados previamente, asegurando que las rutas y la codificación de caracteres (`UTF-8`) fueran interpretadas correctamente por el cliente de PostgreSQL (`psql`).

---

## 2. Flujo de ejecución del script (carga_datos.sh)
El script automatiza los siguientes pasos de manera secuencial:
1. **Verificación de Conexión:** Comprueba la disponibilidad del servidor de base de datos local o en contenedor.
2. **Inyección por Órdenes de Dependencia:**
   * Carga de tablas base (`Persona`).
   * Carga de entidades derivadas (`Cliente`, `Empleado`, `Proveedores`).
   * Carga de catálogos y atributos multivaluados (`PersonaTeléfono`, `ClienteCorreo`, `ProveedorCorreo`, `Productos`, `Categoria`).
   * Carga de operaciones y transacciones (`Inventario`, `Surtir`, `Venta`, `Incluir`).
3. **Validación de restricciones:** Confirma que ningún registro viole los dominios (`dm_monto`, `dm_email`, `dm_telefono`) ni las restricciones `CHECK` y `NOT NULL`.

---

## 3. Evidencias de ejecución

<img width="1366" height="693" alt="E6" src="https://github.com/user-attachments/assets/647bfb6f-d3b9-4c8e-a4cd-975e031cdc5a" />

<img width="1354" height="701" alt="E7" src="https://github.com/user-attachments/assets/fdaa8850-0494-46d6-aba4-8b3e147861ad" />

<img width="1066" height="686" alt="E8" src="https://github.com/user-attachments/assets/08f78b62-ecf4-477b-a6f9-98bb1a264bd3" />

<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/925f6ce4-fd2e-44d8-bf90-e6e7ba7f8b8b" />

---

## 4. Script de Carga Automatizada (carga_datos.sh)


```bash
#!/usr/bin/env bash
set -e

# 5. Sin credenciales adentro: la conexión se arma con variables de entorno
DB_HOST="${PGHOST:-localhost}"
DB_PORT="${PGPORT:-5432}"
DB_USER="${PGUSER:-postgres}"
DB_NAME="${PGDATABASE:-postgres}"

# Ruta absoluta exacta entre comillas para que Git Bash respete el espacio en "Program Files"
PSQL="/c/Program Files/PostgreSQL/18/bin/psql.exe"

echo "=================================================="
echo "Iniciando carga masiva del Supermercado"
echo "Servidor: ${DB_HOST}:${DB_PORT} | Base: ${DB_NAME}"
echo "=================================================="

TMP_SQL=$(mktemp)

cat << 'EOF' > "$TMP_SQL"
-- 3. Todo o nada: si algo falla, se revierte toda la transacción
BEGIN;

-- 1. Repetible: TRUNCATE con CASCADE limpia todo respetando dependencias sin duplicar
TRUNCATE TABLE 
    Incluir, Venta, Surtir, Inventario, Categoria, 
    Productos, ProveedorCorreo, ClienteCorreo, PersonaTelefono, 
    Proveedores, Empleado, Cliente, Persona 
RESTART IDENTITY CASCADE;

-- 2. Ordenado: Primero tablas independientes, luego dependientes (con codificación WIN1252 para evitar errores de acentos)
\copy Persona(Identificador, PrimerNombre, SegundoNombre, ApellidoPaterno, ApellidoMaterno, TipoPersona) FROM 'datos/persona.csv' WITH (FORMAT csv, HEADER true, ENCODING 'WIN1252');
\copy Cliente(Identificador) FROM 'datos/cliente.csv' WITH (FORMAT csv, HEADER true, ENCODING 'WIN1252');
\copy Empleado(Identificador, Puesto) FROM 'datos/empleado.csv' WITH (FORMAT csv, HEADER true, ENCODING 'WIN1252');
\copy Proveedores(Identificador, Calle, Ciudad, Num_Calle) FROM 'datos/proveedores.csv' WITH (FORMAT csv, HEADER true, ENCODING 'UTF8');
\copy PersonaTelefono(Identificador, Telefono) FROM 'datos/personatelefono.csv' WITH (FORMAT csv, HEADER true, ENCODING 'WIN1252');
\copy ClienteCorreo(Identificador, Correo) FROM 'datos/clientecorreo.csv' WITH (FORMAT csv, HEADER true, ENCODING 'WIN1252');
\copy ProveedorCorreo(Identificador, Correo) FROM 'datos/proveedorcorreo.csv' WITH (FORMAT csv, HEADER true, ENCODING 'WIN1252');
\copy Productos(Cod_Barra, Nombre, Marca, Precio) FROM 'datos/productos.csv' WITH (FORMAT csv, HEADER true, ENCODING 'WIN1252');
\copy Categoria(Cod_Barra, Identificador, Nombre) FROM 'datos/categoria.csv' WITH (FORMAT csv, HEADER true, ENCODING 'WIN1252');
\copy Inventario(Cod_Barra, Fecha_Actu, Cantidad) FROM 'datos/inventario.csv' WITH (FORMAT csv, HEADER true, ENCODING 'WIN1252');
\copy Surtir(Cod_Barra, Identificador) FROM 'datos/surtir.csv' WITH (FORMAT csv, HEADER true, ENCODING 'WIN1252');
\copy Venta(Ticket, Identificador_cliente, Identificador_empleado, Total, Dia, Mes, Anio, Empleadoventas) FROM 'datos/venta.csv' WITH (FORMAT csv, HEADER true, ENCODING 'WIN1252');
\copy Incluir(Ticket, Cod_Barra) FROM 'datos/incluir.csv' WITH (FORMAT csv, HEADER true, ENCODING 'WIN1252');

COMMIT;
EOF

# Ejecución eficiente con psql utilizando comillas dobles en la variable
PGPASSWORD="${PGPASSWORD}" "$PSQL" -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" -d "$DB_NAME" -f "$TMP_SQL"
rm -f "$TMP_SQL"

echo ""
echo "=== 4. Conteo de registros insertados por tabla ==="
PGPASSWORD="${PGPASSWORD}" "$PSQL" -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" -d "$DB_NAME" -c "
SELECT 'Persona' AS tabla, COUNT(*) FROM Persona
UNION ALL SELECT 'Cliente', COUNT(*) FROM Cliente
UNION ALL SELECT 'Productos', COUNT(*) FROM Productos
UNION ALL SELECT 'Venta', COUNT(*) FROM Venta;
"
