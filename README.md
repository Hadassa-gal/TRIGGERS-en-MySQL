# **Taller de Triggers en MySQL**

## 📌 **Objetivo**

En este taller, aprenderás a utilizar **Triggers** en MySQL a través de casos prácticos. Implementarás triggers para validaciones, auditoría de cambios y registros automáticos.

------

**TABLES & INSERTS**

```sql
CREATE TABLE productos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50),
    stock INT
);

CREATE TABLE ventas (
    id INT PRIMARY KEY AUTO_INCREMENT,
    id_producto INT,
    cantidad INT,
    FOREIGN KEY (id_producto) REFERENCES productos(id)
);
```

## 

## **🔹 Caso 1: Control de Stock de Productos**

### **Escenario:**

Una tienda en línea necesita asegurarse de que los clientes no puedan comprar más unidades de un producto del stock disponible. Si intentan hacerlo, la compra debe **bloquearse**.

### **Tarea:**

1. Crear las tablas `productos` y `ventas`.
2. Implementar un trigger `BEFORE INSERT` para evitar ventas con cantidad mayor al stock disponible.
3. Probar el trigger.

```sql
DELIMITER //

CREATE TRIGGER before_venta_insert
BEFORE INSERT ON ventas
FOR EACH ROW
BEGIN
	DECLARE stock_actual INT;
	SELECT stock INTO stock_actual FROM productos WHERE id = NEW.id_producto;
    IF NEW.cantidad > stock_actual THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'LA cantidad ingresada a comprar supera el stock';
    END IF;
END //
DELIMITER ;

INSERT INTO productos (nombre, stock) VALUES ('Teclado', 10);

INSERT INTO productos (nombre, stock) VALUES ('Monitor', 5);

```

![](https://media.discordapp.net/attachments/1337463162940817490/1392195468091457660/image.png?ex=686ea691&is=686d5511&hm=584c1a171ddcefed3f441f66b8bb0bd6bdc7058e6f34f09d0ebaa81a7c47ff0f&=&format=webp&quality=lossless)

![](https://media.discordapp.net/attachments/1337463162940817490/1392195721910026334/image.png?ex=686ea6cd&is=686d554d&hm=d569a23ad821f2bd939694fe0fa1841761f6d990eb7203169afee438b106f587&=&format=webp&quality=lossless)

## **🔹 Caso 2: Registro Automático de Cambios en Salarios**

### **Escenario:**

La empresa **TechCorp** desea mantener un registro histórico de todos los cambios de salario de sus empleados.

### **Tarea:**

1. Crear las tablas `empleados` y `historial_salarios`.
2. Implementar un trigger `BEFORE UPDATE` que registre cualquier cambio en el salario.
3. Probar el trigger.

```sql
DELIMITER //
CREATE TRIGGER before_empleados_update
BEFORE UPDATE ON empleados
FOR EACH ROW
BEGIN
	INSERT INTO historial_salarios (id_empleado, salario_anterior, salario_nuevo)
    VALUES (OLD.id, OLD.salario, NEW.salario);
END //
DELIMITER ;

INSERT INTO empleados (nombre, salario) VALUES ('Ana López', 2500.00);
INSERT INTO empleados (nombre, salario) VALUES ('Carlos Pérez', 3000.00);
INSERT INTO empleados (nombre, salario) VALUES ('Lucía Torres', 2800.00);

UPDATE empleados SET salario = 2700.00 WHERE id = 1;
UPDATE empleados SET salario = 3200.00 WHERE id = 2;
UPDATE empleados SET salario = 2850.00 WHERE id = 3;
```

![](https://media.discordapp.net/attachments/1337463162940817490/1392199834517114961/image.png?ex=686eaaa2&is=686d5922&hm=eea89e6b55845943f1896a863ba416afcd8984327ece03069c531c2514ba3b48&=&format=webp&quality=lossless)

![](https://media.discordapp.net/attachments/1337463162940817490/1392199980911165470/image.png?ex=686eaac5&is=686d5945&hm=af131925e997cb6254f28660ba4bf2e1f0b0754fbeaa88f1b462316f5d81b500&=&format=webp&quality=lossless)

## **🔹 Caso 3: Registro de Eliminaciones en Auditoría**

### **Escenario:**

La empresa **DataSecure** quiere registrar toda eliminación de clientes en una tabla de auditoría para evitar pérdidas accidentales de datos.

### **Tarea:**

1. Crear las tablas `clientes` y `clientes_auditoria`.
2. Implementar un trigger `AFTER DELETE` para registrar los clientes eliminados.
3. Probar el trigger.

```sql
DELIMITER //
CREATE TRIGGER historial_clientes
AFTER DELETE ON clientes
FOR EACH ROW
BEGIN
	INSERT INTO clientes_auditoria (id_cliente, nombre, email)
    VALUES (OLD.id, OLD.nombre, OLD.email);
END //
DELIMITER ;

INSERT INTO clientes_auditoria (id_cliente, nombre, email)
VALUES (1, 'Juan Pérez', 'juan.perez@example.com');

INSERT INTO clientes_auditoria (id_cliente, nombre, email)
VALUES (2, 'María Gómez', 'maria.gomez@example.com');

INSERT INTO clientes_auditoria (id_cliente, nombre, email)
VALUES (3, 'Luis Martínez', 'luis.martinez@example.com');

INSERT INTO clientes_auditoria (id_cliente, nombre, email)
VALUES (4, 'Ana Fernández', 'ana.fernandez@example.com');

INSERT INTO clientes_auditoria (id_cliente, nombre, email)
VALUES (5, 'Carlos Ruiz', 'carlos.ruiz@example.com');

DELETE FROM clientes WHERE id = 1;
DELETE FROM clientes WHERE id = 2;
DELETE FROM clientes WHERE id = 3;
DELETE FROM clientes WHERE id = 4;
DELETE FROM clientes WHERE id = 5;
```

![](https://media.discordapp.net/attachments/1337463162940817490/1392204499493584917/image.png?ex=686eaefa&is=686d5d7a&hm=86a58ad1b7762ad6dd0ed1bb5a3e2e783663cab8da2456a30bce0e460fa90ebc&=&format=webp&quality=lossless&width=698&height=860)

![](https://media.discordapp.net/attachments/1337463162940817490/1392204836379955240/image.png?ex=686eaf4a&is=686d5dca&hm=51a4e6fcb848ed7ffd5f21411a2a838168f21c53f410b836e04f3104ceddf840&=&format=webp&quality=lossless)

## **🔹 Caso 4: Restricción de Eliminación de Pedidos Pendientes**

### **Escenario:**

En un sistema de ventas, no se debe permitir eliminar pedidos que aún están **pendientes**.

### **Tarea:**

1. Crear las tablas `pedidos`.
2. Implementar un trigger `BEFORE DELETE` para evitar la eliminación de pedidos pendientes.
3. Probar el trigger.

```sql
CREATE TABLE pedidos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente VARCHAR(100),
    estado ENUM('pendiente', 'completado')
);
```

