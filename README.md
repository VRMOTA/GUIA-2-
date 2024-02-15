DROP DATABASE IF EXISTS EssenZial;

CREATE DATABASE IF NOT EXISTS EssenZial CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;


USE EssenZial;

-- Tabla Clientes
CREATE TABLE IF NOT EXISTS tb_clientes (
    id_cliente INT AUTO_INCREMENT PRIMARY KEY,
    nombre_cliente VARCHAR(255) NOT NULL,
    apellido_cliente VARCHAR(255) NOT NULL,
    correo_cliente VARCHAR(255) UNIQUE,
    telefono_cliente VARCHAR(20) UNIQUE,
    clave_cliente VARCHAR(255) NOT NULL ,
    estado_cliente BOOLEAN DEFAULT TRUE ,
    INDEX(correo_cliente, telefono_cliente) 
) ENGINE=INNODB;



CREATE TABLE IF NOT EXISTS tb_direcciones (
    id_direccion INT AUTO_INCREMENT PRIMARY KEY,
    nombre_direccion VARCHAR(255) NOT NULL,
    direccion_cliente VARCHAR(255) NOT NULL,
    telefono_cliente VARCHAR(20) UNIQUE,
    codigo_postal VARCHAR(5) NOT NULL,
    instrucciones_entrega VARCHAR(100) NOT NULL,
    id_cliente INT NOT NULL,
     FOREIGN KEY (id_cliente) REFERENCES tb_clientes(id_cliente),
    INDEX(direccion_cliente, nombre_direccion) 
) ENGINE=INNODB;


-- Tabla Administradores
CREATE TABLE IF NOT EXISTS tb_admins (
    id_admin INT AUTO_INCREMENT PRIMARY KEY,
    nombre_admin VARCHAR(255 )NOT NULL,
    apellido_admin VARCHAR(255) NOT NULL,
    correo_admin VARCHAR(255) UNIQUE,
    clave_admin VARCHAR(255) NOT NULL
) ENGINE=InnoDB;

-- Tabla Categorías
CREATE TABLE IF NOT EXISTS tb_categorias (
    id_categoria INT AUTO_INCREMENT PRIMARY KEY,
    nombre_categoria VARCHAR(255) NOT NULL, 
    imagen_categoria VARCHAR(100) NOT NULL
) ENGINE=InnoDB;


-- Tabla Marcas
CREATE TABLE IF NOT EXISTS tb_marcas (
    id_marca INT AUTO_INCREMENT PRIMARY KEY,
    nombre_marca VARCHAR(255) UNIQUE NOT NULL,
    imagen_marca VARCHAR(100) NOT NULL
) ENGINE=InnoDB;

-- Tabla Olores
CREATE TABLE IF NOT EXISTS tb_olores (
    id_olor INT AUTO_INCREMENT PRIMARY KEY,
    nombre_olor VARCHAR(255) UNIQUE, 
    imagen_olor VARCHAR(100) NOT NULL
) ENGINE=INNODB;


-- Tabla Inventarios
CREATE TABLE IF NOT EXISTS tb_inventarios (
    id_inventario INT AUTO_INCREMENT PRIMARY KEY,
    nombre_inventario VARCHAR(255) NOT NULL,
    cantidad_inventario INT NOT NULL,
    descripcion_inventario TEXT NOT NULL,
    precio_inventario DECIMAL(10, 2) NOT NULL,
    imagen_producto VARCHAR(100) NOT NULL,
    cantidad_descuento FLOAT NOT NULL,
    descripcion_descuento TEXT,
    fecha_inicio_descuento DATE,
    estado_descuento BOOLEAN ,
    fecha_fin_descuento DATE,
    id_olor INT NOT NULL,
    id_categoria INT NOT NULL,
    id_marca INT NOT NULL,
    FOREIGN KEY (id_olor) REFERENCES tb_olores(id_olor),
    FOREIGN KEY (id_categoria) REFERENCES tb_categorias(id_categoria),
    FOREIGN KEY (id_marca) REFERENCES tb_marcas(id_marca)
) ENGINE=InnoDB;

-- Tabla Imagenes
CREATE TABLE IF NOT EXISTS tb_imagenes (
    id_imagen INT AUTO_INCREMENT PRIMARY KEY,
    ruta_imagen VARCHAR(100) NOT NULL, 
    id_inventario INT NOT NULL,
   FOREIGN KEY (id_inventario) REFERENCES tb_inventarios(id_inventario)
) ENGINE=InnoDB;

-- Tabla Pedidos
CREATE TABLE IF NOT EXISTS tb_pedidos (
    id_pedido INT AUTO_INCREMENT PRIMARY KEY,
    total_pago DECIMAL(10,2),
    numero_pedido VARCHAR(10) NOT NULL UNIQUE,
    fecha_pedido DATE NOT NULL, 
    estado_pedido VARCHAR(250),
    tipo_pago BOOLEAN DEFAULT TRUE,
    id_cliente INT NOT NULL,
    id_direccion INT NOT NULL, 
    FOREIGN KEY (id_cliente) REFERENCES tb_clientes(id_cliente), 
    FOREIGN KEY (id_direccion) REFERENCES tb_direcciones(id_direccion)
) ENGINE=InnoDB;

-- Tabla Detalle Pedido
CREATE TABLE IF NOT EXISTS tb_detalle_pedido (
    id_detalle_pedido INT AUTO_INCREMENT PRIMARY KEY,
    cantidad_producto INT NOT NULL,
    costo_actual DECIMAL(10, 2) NOT NULL, 
    id_pedido INT NOT NULL, 
    id_inventario INT NOT NULL,
    FOREIGN KEY (id_pedido) REFERENCES tb_pedidos(id_pedido),
    FOREIGN KEY (id_inventario) REFERENCES tb_inventarios(id_inventario)
) ENGINE=INNODB;

-- Tabla Valoraciones
CREATE TABLE IF NOT EXISTS tb_valoraciones (
    id_valoracion INT AUTO_INCREMENT PRIMARY KEY,
    calificacion_producto INT NOT NULL,
    comentario_producto VARCHAR(250) NOT NULL,
    fecha_valoracion DATE NOT NULL,
    estado_comentario BOOLEAN DEFAULT TRUE,
    id_detalle_pedido INT NOT NULL,
    id_cliente INT NOT NULL,
    FOREIGN KEY (id_cliente) REFERENCES tb_clientes(id_cliente),
    FOREIGN KEY (id_detalle_pedido) REFERENCES tb_detalle_pedido(id_detalle_pedido)
) ENGINE=INNODB;



DELIMITER //

CREATE TRIGGER actualizar_inventario_despues_de_pedido
AFTER INSERT ON tb_detalle_pedido
FOR EACH ROW
BEGIN
    DECLARE cantidad_pedido INT;
    DECLARE id_producto INT;

    -- Obtener la cantidad de productos y el ID del producto del pedido insertado
    SELECT cantidad_producto, id_inventario INTO cantidad_pedido, id_producto
    FROM tb_detalle_pedido
    WHERE id_detalle_pedido = NEW.id_detalle_pedido;

    -- Restar la cantidad de productos pedidos del inventario
    UPDATE tb_inventarios
    SET cantidad_inventario = cantidad_inventario - cantidad_pedido
    WHERE id_inventario = id_producto;
END;
//

DELIMITER ;


DELIMITER //

CREATE PROCEDURE agregar_cliente(
    IN p_nombre_cliente VARCHAR(255),
    IN p_apellido_cliente VARCHAR(255),
    IN p_correo_cliente VARCHAR(255),
    IN p_telefono_cliente VARCHAR(20),
    IN p_clave_cliente VARCHAR(255)
)
BEGIN
    INSERT INTO tb_clientes(nombre_cliente, apellido_cliente, correo_cliente, telefono_cliente, clave_cliente)
    VALUES(p_nombre_cliente, p_apellido_cliente, p_correo_cliente, p_telefono_cliente, p_clave_cliente);
END //

DELIMITER ;


-- Agregar datos de clientes
INSERT INTO tb_clientes (nombre_cliente, apellido_cliente, correo_cliente, telefono_cliente, clave_cliente)
VALUES 
    ('María', 'González', 'maria@example.com', '123456789', 'clave123'),
    ('Juan', 'Martínez', 'juan@example.com', '987654321', 'clave456');

-- Agregar datos de direcciones
INSERT INTO tb_direcciones (nombre_direccion, direccion_cliente, telefono_cliente, codigo_postal, instrucciones_entrega, id_cliente)
VALUES 
    ('Casa', 'Calle 123, Ciudad', '123456789', '12345', 'Frente al parque', 1),
    ('Oficina', 'Av. Principal 456, Ciudad', '987654321', '54321', 'Edificio 3, Piso 5', 2);

-- Agregar datos de administradores
INSERT INTO tb_admins (nombre_admin, apellido_admin, correo_admin, clave_admin)
VALUES 
    ('Admin', 'Principal', 'admin@example.com', 'admin123');

-- Agregar datos de categorías
INSERT INTO tb_categorias (nombre_categoria, imagen_categoria)
VALUES 
    ('Fragancias', 'fragancias.jpg');

-- Agregar datos de marcas
INSERT INTO tb_marcas (nombre_marca, imagen_marca)
VALUES 
    ('MarcaA', 'marcaA.jpg'),
    ('MarcaB', 'marcaB.jpg');

-- Agregar datos de olores
INSERT INTO tb_olores (nombre_olor, imagen_olor)
VALUES 
    ('Floral', 'floral.jpg'),
    ('Cítrico', 'citrico.jpg');

-- Agregar datos de inventarios (productos - perfumes)
INSERT INTO tb_inventarios (nombre_inventario, cantidad_inventario, descripcion_inventario, precio_inventario, imagen_producto, cantidad_descuento, descripcion_descuento, fecha_inicio_descuento, estado_descuento, fecha_fin_descuento, id_olor, id_categoria, id_marca)
VALUES 
    ('Perfume A', 100, 'Fragancia floral suave', 50.00, 'perfumeA.jpg', 0.1, 'Descuento de lanzamiento', '2024-02-01', TRUE, '2024-02-28', 1, 1, 1),
    ('Perfume B', 80, 'Fragancia cítrica refrescante', 45.00, 'perfumeB.jpg', 0, NULL, NULL, FALSE, NULL, 2, 1, 2);

-- Agregar datos de imágenes
INSERT INTO tb_imagenes (ruta_imagen, id_inventario)
VALUES 
    ('imagen1.jpg', 1),
    ('imagen2.jpg', 2);

-- Agregar datos de pedidos
INSERT INTO tb_pedidos (total_pago, numero_pedido, fecha_pedido, estado_pedido, tipo_pago, id_cliente, id_direccion)
VALUES 
    (50.00, 'P123', '2024-02-10', 'En proceso', TRUE, 1, 1),
    (45.00, 'P124', '2024-02-12', 'Entregado', TRUE, 2, 2);

-- Agregar datos de detalle de pedidos
INSERT INTO tb_detalle_pedido (cantidad_producto, costo_actual, id_pedido, id_inventario)
VALUES 
    (1, 50.00, 1, 1),
    (1, 45.00, 2, 2);

-- Agregar datos de valoraciones
INSERT INTO tb_valoraciones (calificacion_producto, comentario_producto, fecha_valoracion, estado_comentario, id_detalle_pedido, id_cliente)
VALUES 
    (5, '¡Me encantó este perfume!', '2024-02-11', TRUE, 1, 1),
    (4, 'Buena fragancia, pero el envase podría mejorar', '2024-02-13', TRUE, 2, 2),
    (3, 'No cumplió mis expectativas', '2024-02-14', TRUE, 1, 2),
    (5, 'Excelente fragancia, volvería a comprar', '2024-02-15', TRUE, 2, 1),
     (4, 'Buena fragancia, pero no dura mucho tiempo', '2024-02-16', TRUE, 1, 1),
    (2, 'No me gustó el olor, no lo recomendaría', '2024-02-17', TRUE, 2, 2),
    (5, 'Increíble aroma, me encanta', '2024-02-18', TRUE, 1, 2),
    (3, 'El perfume es regular, esperaba más', '2024-02-19', TRUE, 2, 1),
    (4, 'Buena relación calidad-precio', '2024-02-20', TRUE, 1, 1);



SELECT * FROM tb_valoraciones;

DELIMITER //

CREATE FUNCTION calcular_promedio_calificacion_producto(id_inventario_param INT) RETURNS DECIMAL(5,2)
BEGIN
    DECLARE promedio DECIMAL(5,2);

    SELECT AVG(calificacion_producto) INTO promedio
    FROM tb_valoraciones
    INNER JOIN tb_detalle_pedido ON tb_valoraciones.id_detalle_pedido = tb_detalle_pedido.id_detalle_pedido
    WHERE tb_detalle_pedido.id_inventario = id_inventario_param;

    RETURN promedio;
END //

DELIMITER ;

SELECT calcular_promedio_calificacion_producto(1);


DELIMITER //

CREATE TRIGGER actualizar_inventario_despues_de_pedido
AFTER INSERT ON tb_detalle_pedido
FOR EACH ROW
BEGIN
    DECLARE cantidad_pedido INT;
    DECLARE id_producto INT;

    -- Obtener la cantidad de productos y el ID del producto del pedido insertado
    SELECT cantidad_producto, id_inventario INTO cantidad_pedido, id_producto
    FROM tb_detalle_pedido
    WHERE id_detalle_pedido = NEW.id_detalle_pedido;

    -- Restar la cantidad de productos pedidos del inventario
    UPDATE tb_inventarios
    SET cantidad_inventario = cantidad_inventario - cantidad_pedido
    WHERE id_inventario = id_producto;
END;
//

DELIMITER ;


DELIMITER //

CREATE PROCEDURE agregar_cliente(
    IN p_nombre_cliente VARCHAR(255),
    IN p_apellido_cliente VARCHAR(255),
    IN p_correo_cliente VARCHAR(255),
    IN p_telefono_cliente VARCHAR(20),
    IN p_clave_cliente VARCHAR(255)
)
BEGIN
    INSERT INTO tb_clientes(nombre_cliente, apellido_cliente, correo_cliente, telefono_cliente, clave_cliente)
    VALUES(p_nombre_cliente, p_apellido_cliente, p_correo_cliente, p_telefono_cliente, p_clave_cliente);
END //

DELIMITER ;

