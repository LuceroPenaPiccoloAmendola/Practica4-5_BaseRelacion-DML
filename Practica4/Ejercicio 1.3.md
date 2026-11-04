# Ejercicio 1.3: Documentación de la Fuente de Datos del Proyecto


### Bitácora de búsqueda:
* **Portales consultados:** Datos Abiertos del Gobierno de México (datos.gob.mx), Repositorio del INEGI y Kaggle datasets.
* **Términos de búsqueda utilizados:** `supermercado ventas`, `abarrotes punto de venta`, `ticket venta inventario mexico`, `retail transactions Mexico`.
* **Resultado:** Se encontraron datos agregados de consumo e índices macroeconómicos, pero no existen conjuntos de datos públicos abiertos que contengan transacciones granulares a nivel de ticket con la estructura relacional de este proyecto (asociación de personas, teléfonos múltiples, correos múltiples, control de surtido de proveedores y renglones de tickets `Incluir`).

### Conclusión:
Se generó un conjunto de datos sintéticos mediante un script en que respeta todas las restricciones de integridad relacional del esquema. Se generaron más de 200 renglones por tabla principal para garantizar consultas expresivas.

---


* **Formato original:** Archivos planos de valores separados por coma (`CSV`).
* **Codificación:** UTF-8 sin BOM.
* **Volumen y registros:**
  * `persona.csv`: 300 renglones
  * `cliente.csv`: 200 renglones
  * `empleado.csv`: 100 renglones
  * `proveedores.csv`: 50 renglones
  * `personatelefono.csv`: 350 renglones
  * `clientecorreo.csv`: 200 renglones
  * `proveedorcorreo.csv`: 50 renglones
  * `productos.csv`: 500 renglones
  * `categoria.csv`: 500 renglones
  * `inventario.csv`: 500 renglones
  * `surtir.csv`: 600 renglones
  * `venta.csv`: 1000 renglones
  * `incluir.csv`: 1700 renglones

---



| Campo del CSV | Tabla Destino | Columna Destino | Tratamiento de Limpieza y Reglas Aplicadas |
| :--- | :--- | :--- | :--- |
| `identificador` | `Persona` | `Identificador` | Clave primaria. Se eliminaron espacios y caracteres especiales. |
| `primer_nombre` | `Persona` | `PrimerNombre` | Normalización de acentos y Mayúscula inicial. |
| `segundo_nombre`| `Persona` | `SegundoNombre` | Manejo explícito de valores `NULL` vacíos. |
| `apellido_p` | `Persona` | `ApellidoPaterno` | Normalización de texto sin caracteres corruptos. |
| `apellido_m` | `Persona` | `ApellidoMaterno` | Manejo de nulos. |
| `tipo_persona` | `Persona` | `TipoPersona` | Restringido a lista permitida: `Cliente`, `Empleado`, `Proveedor`. |
| `puesto` | `Empleado` | `Puesto` | Asignación de puestos reales (`Cajero`, `Gerente`, `Surtidor`). |
| `calle`, `num` | `Proveedores` | `Calle`, `Num_Calle` | Dirección física sin duplicados. |
| `telefono` | `PersonaTeléfono`| `Teléfono` | Relleno con ceros para garantizar exactamente 10 dígitos. |
| `correo` | `ClienteCorreo` | `Correo` | Filtrado y validación de expresiones regulares de correo. |
| `cod_barra` | `Productos` | `Cod_Barra` | Formato EAN-13 único de 13 dígitos de longitud fija. |
| `precio` | `Productos` | `Precio` | Monto positivo mayor a cero (NUMERIC decimal, no float). |
| `ticket` | `Venta` | `Ticket` | Identificador de ticket consecutivo rellenado con ceros (`T-00001`). |
| `total` | `Venta` | `Total` | Reconciliación con la suma del detalle de productos. |
| `dia`, `mes`, `anio`| `Venta` | `Dia`, `Mes`, `Año` | Fechas descompuestas válidas (Días entre 1 y 31, Meses 1 a 12). |

---

---

## Automatización de carga con python (Script de generación sintética)

Para cumplir con la regla de que la carga de datos no se realiza a mano y debe poder repetirse de manera automatizada, se desarrolló un script en **Python** que genera los archivos `CSV` asegurando la integridad referencial y respetando los dominios del sistema.

### ¿Cómo funciona el script?
1. **Generación con librerías estándar:** Utiliza librerías como csv y módulos de aleatoriedad controlada para poblar las 13 tablas relacionales manteniendo la consistencia entre llaves primarias y foráneas (por ejemplo, asegurando que un ticket en Venta solo apunte a clientes y empleados existentes).
2. **Formato estricto:** Rellena identificadores con ceros a la izquierda, formatea montos decimales con dos cifras y valida que los correos y teléfonos cumplan con las expresiones regulares definidas en la base de datos.
3. **Ejecución automatizada:** Con un solo comando en la terminal, se recrean y pueblan todos los datos de prueba listos para las consultas del sistema.

```python
import csv
import random

# Ejemplo resumido de la lógica de generación para nombres
def generar_personas(num_registros=300):
    nombres = ["Ana", "Carlos", "María", "José", "Sofía", "Luis", "Fernanda", "David", "Valeria", "Jorge"]
    apellidos_p = ["González", "Rodríguez", "López", "Martínez", "Pérez", "Gómez", "Sánchez", "Ramírez"]
    apellidos_m = ["Cruz", "Flores", "García", "Morales", "Ortiz", "Rivera", "Gutiérrez", "Jiménez"]
    tipos = ["Cliente", "Empleado", "Proveedor"]
    
    with open('persona.csv', mode='w', newline='', encoding='utf-8') as archivo:
        writer = csv.writer(archivo)
        for i in range(1, num_registros + 1):
            identificador = f"ID-{i:04d}"
            primer_nombre = random.choice(nombres)
            segundo_nombre = random.choice([random.choice(nombres), ""])
            apellido_paterno = random.choice(apellidos_p)
            apellido_materno = random.choice(apellidos_m)
            tipo_persona = random.choice(tipos)
            
            writer.writerow([identificador, primer_nombre, segundo_nombre, apellido_paterno, apellido_materno, tipo_persona])

if __name__ == "__main__":
    generar_personas()
    print("Datos sintéticos de personas generados exitosamente.")
