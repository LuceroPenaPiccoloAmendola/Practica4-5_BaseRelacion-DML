# Ejercicio 1.5: Pruebas de Restricciones de Dominio

En este documento se registran diez casos de prueba diseñados para evaluar la robustez del esquema de bases de datos del supermercado. Cada prueba intenta vulnerar una restricción específica (`NOT NULL`, `UNIQUE`, `CHECK`, etc.) para verificar que el Sistema Gestor de Base de Datos (SGBD) rechace la operación y proteja la integridad de la información conforme a las reglas del negocio.

---

## 1. Casos de Prueba de Restricciones

### Caso 1: Violación de `NOT NULL` en Nombre de Persona
* **Operación intentada:** 
  ```sql
  INSERT INTO Persona (Identificador, PrimerNombre, ApellidoPaterno, TipoPersona) 
  VALUES ('ID-999', NULL, 'Pérez', 'Cliente');
  ```
  <img width="1366" height="568" alt="image" src="https://github.com/user-attachments/assets/cfed54a0-fd7f-4d90-be5c-2e45ae4cfeb2" />

---

### Caso 2: Violación de numero negativo
* **Operación intentada:**
 ```sql
  INSERT INTO Productos (Cod_Barra, Nombre, Marca, Precio) 
VALUES ('750199999999', 'Articulo Invalido', 'Test', -15.50);
```
<img width="1366" height="587" alt="image" src="https://github.com/user-attachments/assets/4056481e-889b-4012-a2f8-87feff2a08b3" />

---

### Caso 3: Violación de check en inventario
* **Operación intentada:**
   ```sql
  INSERT INTO Inventario (Cod_Barra, Fecha_Actu, Cantidad) 
VALUES ('750100010001', '2026-11-04', -5);


<img width="1366" height="582" alt="image" src="https://github.com/user-attachments/assets/2589ec04-310e-4963-ad9c-15119f541cc1" />

---

### Caso 4: Violación de numero de telefono
* **Operación intentada:**
 ```sql
INSERT INTO PersonaTeléfono (Identificador, Teléfono) 
VALUES ('ID-0001', '5512345556');
 ```

<img width="1366" height="564" alt="image" src="https://github.com/user-attachments/assets/31d09acc-f09b-4ef2-b858-65899ec9bd4a" />

---

### Caso 5: Violación llave primaria
* **Operación intentada:**
 ```sql
INSERT INTO Venta (Ticket, Identificador_cliente, Identificador_empleado, Total, Dia, Mes, Año) 
VALUES ('10001', '100', '200', 100.00, 20, 11, 2026);
```
<img width="1366" height="557" alt="image" src="https://github.com/user-attachments/assets/39d519ba-ea15-4c8b-83c5-b9324af3755f" />

---

### Caso 6: Violación categoría
* **Operación intentada:**
 ```sql
INSERT INTO Categoria (Cod_Barra, Identificador, Nombre) 
VALUES ('750999999999', 'CAT-01', 'Lácteos');
 ```
<img width="1366" height="581" alt="image" src="https://github.com/user-attachments/assets/8871d957-1c90-4c11-afb7-3f973bdaf792" />

---

### Caso 7: Violación inventario
* **Operación intentada:**
 ```sql
INSERT INTO Inventario (Cod_Barra, Fecha_Actu, Cantidad) 
VALUES ('750888888888', '2026-11-04', 100);
```
<img width="1366" height="600" alt="image" src="https://github.com/user-attachments/assets/f32eb477-8b8e-4f24-b252-bb890a95f371" />

---

### Caso 8: Violación restricción
* **Operación intentada:**
```sql
DELETE FROM Empleado WHERE Identificador = 'ID-EMPLEADO-CON-VENTAS';
```
<img width="1366" height="631" alt="image" src="https://github.com/user-attachments/assets/fd0a1658-2f10-4276-9b6a-a1924ddb3819" />

---

### Caso 9: Violación llave duplicado
* **Operación intentada:**
```sql
INSERT INTO PersonaTelefono (Identificador, Telefono) VALUES ('105', '5561223344');
-- Intentar registrar el mismo teléfono con otro usuario:
INSERT INTO PersonaTelefono (Identificador, Telefono) VALUES ('106', '5561223344');
```
<img width="1366" height="638" alt="image" src="https://github.com/user-attachments/assets/6f0c3ac0-2072-4c2d-88fb-052ae054aeaf" />

---

### Caso 10: Violación año
* **Operación intentada:**
```sql
INSERT INTO Venta (Ticket, Identificador_cliente, Identificador_empleado, Total, Dia, Mes, Anio) 
VALUES ('10001', '48', '228', 450.00, 15, 6, 1995);
```

<img width="1366" height="579" alt="image" src="https://github.com/user-attachments/assets/b282c578-7824-41fa-8f70-665538a0271b" />




