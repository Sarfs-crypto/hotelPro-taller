SET FOREIGN_KEY_CHECKS = 0;

-- Eliminar tablas si existen 
DROP TABLE IF EXISTS reservaciones_servicios;
DROP TABLE IF EXISTS reservas_salones;
DROP TABLE IF EXISTS salones_eventos;
DROP TABLE IF EXISTS eventos;
DROP TABLE IF EXISTS temporadas;
DROP TABLE IF EXISTS programa_fidelizacion;
DROP TABLE IF EXISTS reservaciones;
DROP TABLE IF EXISTS servicios_extras;
DROP TABLE IF EXISTS habitaciones;
DROP TABLE IF EXISTS huespedes;


-- Tabla de huéspedes
CREATE TABLE huespedes (
  dni VARCHAR(20) PRIMARY KEY,
  nombre_completo VARCHAR(100) NOT NULL,
  correo_electronico VARCHAR(100) NOT NULL,
  nacionalidad VARCHAR(50),
  telefono VARCHAR(20),
  preferencias TEXT,
  fecha_registro DATE DEFAULT CURRENT_DATE,
  INDEX idx_correo (correo_electronico),
  INDEX idx_nombre (nombre_completo)
);

-- Tabla de habitaciones
CREATE TABLE habitaciones (
  numero_unico INT PRIMARY KEY,
  tipo_habitacion VARCHAR(50) NOT NULL,
  tarifa_base DECIMAL(10,2) NOT NULL,
  tarifa_actual DECIMAL(10,2) NOT NULL,
  piso INT NOT NULL,
  capacidad INT NOT NULL,
  caracteristicas_especiales TEXT,
  estado ENUM('disponible', 'ocupada', 'mantenimiento', 'reservada') DEFAULT 'disponible',
  INDEX idx_tipo (tipo_habitacion),
  INDEX idx_estado (estado)
);

-- Tabla de temporadas
CREATE TABLE temporadas (
  id_temporada INT PRIMARY KEY AUTO_INCREMENT,
  nombre VARCHAR(50) NOT NULL,
  fecha_inicio DATE NOT NULL,
  fecha_fin DATE NOT NULL,
  multiplicador_tarifa DECIMAL(3,2) NOT NULL DEFAULT 1.0,
  descripcion TEXT,
  INDEX idx_fechas (fecha_inicio, fecha_fin)
);

-- Tabla de servicios extras
CREATE TABLE servicios_extras (
  id_servicio INT PRIMARY KEY AUTO_INCREMENT,
  nombre VARCHAR(100) NOT NULL,
  costo DECIMAL(10,2) NOT NULL,
  descripcion TEXT,
  categoria ENUM('gastronomia', 'transporte', 'bienestar', 'entretenimiento', 'otros'),
  disponible BOOLEAN DEFAULT TRUE,
  INDEX idx_categoria (categoria)
);

-- Tabla de reservaciones
CREATE TABLE reservaciones (
  id_reserva VARCHAR(20) PRIMARY KEY,
  dni_huesped VARCHAR(20) NOT NULL,
  numero_habitacion INT,
  fecha_entrada DATE NOT NULL,
  fecha_de_salida DATE NOT NULL,
  estado_pago ENUM('pendiente', 'pagado', 'cancelado') NOT NULL DEFAULT 'pendiente',
  estado_reserva ENUM('confirmada', 'check-in', 'check-out', 'no-show') DEFAULT 'confirmada',
  numero_huespedes INT NOT NULL,
  tarifa_aplicada DECIMAL(10,2),
  descuento_aplicado DECIMAL(5,2) DEFAULT 0,
  deposito_garantia DECIMAL(10,2) DEFAULT 0,
  fecha_solicitud DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  fecha_checkin DATETIME,
  fecha_checkout DATETIME,
  FOREIGN KEY (dni_huesped) REFERENCES huespedes(dni) ON DELETE CASCADE,
  FOREIGN KEY (numero_habitacion) REFERENCES habitaciones(numero_unico) ON DELETE SET NULL,
  INDEX idx_fechas (fecha_entrada, fecha_de_salida),
  INDEX idx_estado_pago (estado_pago),
  INDEX idx_estado_reserva (estado_reserva)
);

-- Tabla de programa de fidelización
CREATE TABLE programa_fidelizacion (
  dni_huesped VARCHAR(20) PRIMARY KEY,
  puntos_acumulados INT DEFAULT 0,
  nivel ENUM('bronce', 'plata', 'oro', 'platino') DEFAULT 'bronce',
  fecha_inscripcion DATE DEFAULT CURRENT_DATE,
  fecha_ultima_actualizacion DATE DEFAULT CURRENT_DATE,
  total_estancias INT DEFAULT 0,
  total_noches INT DEFAULT 0,
  FOREIGN KEY (dni_huesped) REFERENCES huespedes(dni) ON DELETE CASCADE,
  INDEX idx_nivel (nivel),
  INDEX idx_puntos (puntos_acumulados)
);

-- Tabla de eventos
CREATE TABLE eventos (
  id_evento INT PRIMARY KEY AUTO_INCREMENT,
  nombre_evento VARCHAR(100) NOT NULL,
  tipo_evento ENUM('conferencia', 'boda', 'reunion', 'celebracion', 'corporativo', 'social', 'deportivo', 'cultural') NOT NULL,
  fecha_evento DATE NOT NULL,
  hora_inicio TIME NOT NULL,
  hora_fin TIME NOT NULL,
  numero_asistentes INT NOT NULL,
  descripcion TEXT,
  costo_evento DECIMAL(10,2) NOT NULL,
  estado ENUM('planificado', 'confirmado', 'en_curso', 'completado', 'cancelado') DEFAULT 'planificado',
  id_reserva VARCHAR(20),
  requerimientos_especiales TEXT,
  fecha_creacion DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (id_reserva) REFERENCES reservaciones(id_reserva) ON DELETE SET NULL,
  INDEX idx_fecha_evento (fecha_evento),
  INDEX idx_tipo_evento (tipo_evento),
  INDEX idx_estado (estado)
);

-- Tabla de salones para eventos
CREATE TABLE salones_eventos (
  id_salon INT PRIMARY KEY AUTO_INCREMENT,
  nombre VARCHAR(100) NOT NULL,
  capacidad INT NOT NULL,
  costo_por_hora DECIMAL(10,2) NOT NULL,
  caracteristicas TEXT,
  estado ENUM('disponible', 'ocupado', 'mantenimiento') DEFAULT 'disponible',
  ubicacion VARCHAR(100),
  INDEX idx_capacidad (capacidad),
  INDEX idx_estado (estado)
);

-- Tabla de reservas de salones
CREATE TABLE reservas_salones (
  id_reserva_salon INT PRIMARY KEY AUTO_INCREMENT,
  id_salon INT NOT NULL,
  id_evento INT NOT NULL,
  fecha DATE NOT NULL,
  hora_inicio TIME NOT NULL,
  hora_fin TIME NOT NULL,
  costo_total DECIMAL(10,2) NOT NULL,
  estado ENUM('reservado', 'en_uso', 'finalizado', 'cancelado') DEFAULT 'reservado',
  FOREIGN KEY (id_salon) REFERENCES salones_eventos(id_salon) ON DELETE CASCADE,
  FOREIGN KEY (id_evento) REFERENCES eventos(id_evento) ON DELETE CASCADE,
  INDEX idx_fecha_hora (fecha, hora_inicio),
  INDEX idx_salon_fecha (id_salon, fecha)
);

-- Tabla de servicios de reservas
CREATE TABLE reservaciones_servicios (
  id_registro INT PRIMARY KEY AUTO_INCREMENT,
  id_reserva VARCHAR(20) NOT NULL,
  id_servicio INT NOT NULL,
  fecha_servicio DATE NOT NULL DEFAULT CURRENT_DATE,
  hora_servicio TIME,
  cantidad INT NOT NULL DEFAULT 1,
  costo_unitario DECIMAL(10,2) NOT NULL,
  costo_total DECIMAL(10,2) NOT NULL,
  notas TEXT,
  FOREIGN KEY (id_reserva) REFERENCES reservaciones(id_reserva) ON DELETE CASCADE,
  FOREIGN KEY (id_servicio) REFERENCES servicios_extras(id_servicio) ON DELETE CASCADE,
  INDEX idx_fecha_servicio (fecha_servicio),
  INDEX idx_reserva_servicio (id_reserva, id_servicio)
);



-- Insertar huéspedes
INSERT INTO huespedes (dni, nombre_completo, correo_electronico, nacionalidad, telefono, preferencias) VALUES
('12345678A', 'María González Pérez', 'maria.gonzalez@email.com', 'Española', '+34 600 111 222', 'Vista al mar, piso alto, cama king'),
('87654321B', 'Carlos Rodríguez López', 'carlos.rodriguez@email.com', 'Mexicana', '+52 55 1234 5678', 'Cerca de ascensor, habitación silenciosa'),
('11223344C', 'Ana Martínez Silva', 'ana.martinez@email.com', 'Colombiana', '+57 1 234 5678', 'Suite, jacuzzi, servicio a la habitación'),
('44332211D', 'John Smith Wilson', 'john.smith@email.com', 'Estadounidense', '+1 555 0123', 'Zona de trabajo, internet rápido'),
('55667788E', 'Laura Chen Wong', 'laura.chen@email.com', 'China', '+86 10 5987 6543', 'Desayuno incluido, cerca del spa');

-- Insertar habitaciones
INSERT INTO habitaciones (numero_unico, tipo_habitacion, tarifa_base, tarifa_actual, piso, capacidad, caracteristicas_especiales) VALUES
(101, 'Individual', 80.00, 80.00, 1, 1, 'Vista al jardín, Aire acondicionado, Escritorio'),
(102, 'Individual', 75.00, 75.00, 1, 1, 'Aire acondicionado, Escritorio, WiFi rápido'),
(103, 'Individual', 85.00, 85.00, 1, 1, 'Vista a la piscina, Balcón pequeño'),
(201, 'Doble', 120.00, 120.00, 2, 2, 'Vista al mar, Terraza, Minibar, TV pantalla plana'),
(202, 'Doble', 110.00, 110.00, 2, 2, 'Vista a la ciudad, Minibar, Aire acondicionado'),
(203, 'Doble', 130.00, 130.00, 2, 3, 'Vista al mar, Terraza amplia, Sofá cama'),
(301, 'Suite', 250.00, 250.00, 3, 2, 'Suite presidencial, Jacuzzi, Terraza privada, Sala de estar'),
(302, 'Suite', 200.00, 200.00, 3, 2, 'Suite ejecutiva, Minibar surtido, Zona de trabajo'),
(303, 'Suite', 300.00, 300.00, 3, 4, 'Suite familiar, Dos habitaciones, Cocina pequeña');

-- Insertar temporadas
INSERT INTO temporadas (nombre, fecha_inicio, fecha_fin, multiplicador_tarifa, descripcion) VALUES
('Temporada Baja', '2025-01-01', '2025-03-31', 0.8, 'Periodo de menor demanda'),
('Temporada Media', '2025-04-01', '2025-05-31', 1.0, 'Periodo de demanda regular'),
('Temporada Alta', '2025-06-01', '2025-08-31', 1.5, 'Periodo vacacional - alta demanda'),
('Temporada Especial', '2025-12-15', '2025-12-31', 2.0, 'Navidad y Año Nuevo');

-- Insertar servicios extras
INSERT INTO servicios_extras (nombre, costo, descripcion, categoria) VALUES
('Desayuno buffet', 15.00, 'Desayuno continental completo con variedad internacional', 'gastronomia'),
('Parking cubierto', 10.00, 'Estacionamiento cubierto 24h con seguridad', 'transporte'),
('Spa - Masaje relajante', 50.00, 'Masaje relajante de 60 minutos con aromaterapia', 'bienestar'),
('Tour ciudad guiado', 35.00, 'Recorrido guiado por los principales puntos de la ciudad', 'entretenimiento'),
('Traslado aeropuerto', 30.00, 'Servicio de taxi privado aeropuerto-hotel', 'transporte'),
('Cena romántica', 80.00, 'Cena para dos en terraza privada con chef personal', 'gastronomia'),
('Lavandería express', 25.00, 'Servicio de lavandería en 4 horas', 'otros'),
('Alquiler coche premium', 90.00, 'Alquiler de vehículo premium por día', 'transporte');

-- Insertar programa de fidelización
INSERT INTO programa_fidelizacion (dni_huesped, puntos_acumulados, nivel, total_estancias, total_noches) VALUES
('12345678A', 450, 'oro', 5, 15),
('87654321B', 120, 'plata', 2, 6),
('11223344C', 800, 'platino', 8, 25),
('44332211D', 50, 'bronce', 1, 3),
('55667788E', 300, 'oro', 4, 12);

-- Insertar reservaciones
INSERT INTO reservaciones (id_reserva, dni_huesped, numero_habitacion, fecha_entrada, fecha_de_salida, estado_pago, estado_reserva, numero_huespedes, tarifa_aplicada, deposito_garantia) VALUES
('RES001', '12345678A', 201, '2025-10-30', '2025-11-01', 'pagado', 'check-out', 2, 120.00, 50.00),
('RES002', '87654321B', 102, '2025-10-30', '2025-11-03', 'pendiente', 'confirmada', 1, 75.00, 25.00),
('RES003', '11223344C', 301, '2025-10-30', '2025-11-05', 'pagado', 'check-in', 2, 250.00, 100.00),
('RES004', '44332211D', 202, '2025-11-10', '2025-11-12', 'pendiente', 'confirmada', 2, 110.00, 40.00),
('RES005', '55667788E', 103, '2025-11-15', '2025-11-20', 'pagado', 'confirmada', 1, 85.00, 30.00);

-- Insertar servicios de reservas
INSERT INTO reservaciones_servicios (id_reserva, id_servicio, fecha_servicio, cantidad, costo_unitario, costo_total) VALUES
('RES001', 1, '2025-10-30', 2, 15.00, 30.00),
('RES001', 2, '2025-10-30', 1, 10.00, 10.00),
('RES002', 1, '2025-10-30', 1, 15.00, 15.00),
('RES003', 1, '2025-10-30', 2, 15.00, 30.00),
('RES003', 3, '2025-10-31', 1, 50.00, 50.00),
('RES003', 4, '2025-11-01', 2, 35.00, 70.00);

-- Insertar salones de eventos
INSERT INTO salones_eventos (nombre, capacidad, costo_por_hora, caracteristicas, ubicacion) VALUES
('Salón Principal', 200, 300.00, 'Escenario, equipo de sonido, iluminación profesional', 'Planta Baja - Ala Este'),
('Salón Jardín', 100, 200.00, 'Vista al jardín, terraza exterior, ambiente natural', 'Planta Baja - Jardín'),
('Sala de Juntas Ejecutiva', 30, 100.00, 'Mesa de reuniones, proyector, WiFi empresarial', 'Primera Planta - Ala Oeste'),
('Salón de Fiestas', 150, 250.00, 'Pista de baile, bar privado, iluminación decorativa', 'Segunda Planta - Ala Sur');

-- Insertar eventos
INSERT INTO eventos (nombre_evento, tipo_evento, fecha_evento, hora_inicio, hora_fin, numero_asistentes, descripcion, costo_evento, estado, id_reserva) VALUES
('Conferencia Tech Solutions 2025', 'conferencia', '2025-10-30', '09:00:00', '18:00:00', 150, 'Conferencia anual de tecnología e innovación', 5000.00, 'completado', 'RES003'),
('Boda María y Carlos', 'boda', '2025-10-30', '16:00:00', '23:00:00', 80, 'Ceremonia y recepción de boda', 8000.00, 'completado', 'RES001'),
('Reunión Corporativa Global', 'corporativo', '2025-11-15', '10:00:00', '17:00:00', 25, 'Reunión de directivos internacionales', 1200.00, 'confirmado', 'RES005');

-- Insertar reservas de salones
INSERT INTO reservas_salones (id_salon, id_evento, fecha, hora_inicio, hora_fin, costo_total) VALUES
(1, 1, '2025-10-30', '09:00:00', '18:00:00', 2700.00),
(2, 2, '2025-10-30', '16:00:00', '23:00:00', 1400.00),
(3, 3, '2025-11-15', '10:00:00', '17:00:00', 700.00);

SET FOREIGN_KEY_CHECKS = 1;

SELECT '=== BASE DE DATOS COMPLETA CREADA ===' as info;
SELECT 'Todas las tablas y datos insertados correctamente' as mensaje;



DELIMITER //

-- 1. CrearReserva
DROP PROCEDURE IF EXISTS CrearReserva//
CREATE PROCEDURE CrearReserva(
    IN p_id_reserva VARCHAR(20),
    IN p_dni_huesped VARCHAR(20),
    IN p_tipo_habitacion VARCHAR(50),
    IN p_fecha_entrada DATE,
    IN p_fecha_salida DATE,
    IN p_numero_huespedes INT
)
BEGIN
    DECLARE v_habitacion_disponible INT;
    DECLARE v_tarifa_base DECIMAL(10,2);
    DECLARE v_tarifa_temporada DECIMAL(10,2);
    DECLARE v_multiplicador DECIMAL(3,2);
    DECLARE v_tarifa_final DECIMAL(10,2);
    
    -- Verificar disponibilidad
    SELECT numero_unico, tarifa_base INTO v_habitacion_disponible, v_tarifa_base
    FROM habitaciones 
    WHERE tipo_habitacion = p_tipo_habitacion 
    AND estado = 'disponible'
    AND numero_unico NOT IN (
        SELECT numero_habitacion 
        FROM reservaciones 
        WHERE fecha_entrada <= p_fecha_salida 
        AND fecha_de_salida >= p_fecha_entrada
        AND estado_pago != 'cancelado'
        AND estado_reserva != 'cancelado'
    )
    LIMIT 1;
    
    IF v_habitacion_disponible IS NOT NULL THEN
        -- Obtener multiplicador de temporada
        SELECT COALESCE(multiplicador_tarifa, 1.0) INTO v_multiplicador
        FROM temporadas 
        WHERE p_fecha_entrada BETWEEN fecha_inicio AND fecha_fin
        LIMIT 1;
        
        SET v_tarifa_final = v_tarifa_base * v_multiplicador;
        
        -- Crear reserva
        INSERT INTO reservaciones (id_reserva, dni_huesped, numero_habitacion, fecha_entrada, fecha_de_salida, estado_pago, numero_huespedes, tarifa_aplicada)
        VALUES (p_id_reserva, p_dni_huesped, v_habitacion_disponible, p_fecha_entrada, p_fecha_salida, 'pendiente', p_numero_huespedes, v_tarifa_final);
        
        -- Actualizar estado de habitación
        UPDATE habitaciones SET estado = 'reservada' WHERE numero_unico = v_habitacion_disponible;
        
        SELECT CONCAT('Reserva creada exitosamente. Habitación: ', v_habitacion_disponible, '. Tarifa por noche: €', v_tarifa_final) as mensaje;
    ELSE
        SELECT 'No hay habitaciones disponibles del tipo solicitado para las fechas indicadas' as mensaje;
    END IF;
END //

-- 2. ProcesarCheckIn
DROP PROCEDURE IF EXISTS ProcesarCheckIn//
CREATE PROCEDURE ProcesarCheckIn(
    IN p_id_reserva VARCHAR(20),
    IN p_deposito DECIMAL(10,2)
)
BEGIN
    DECLARE v_estado_reserva VARCHAR(20);
    DECLARE v_numero_habitacion INT;
    DECLARE v_fecha_entrada DATE;
    
    SELECT estado_reserva, numero_habitacion, fecha_entrada 
    INTO v_estado_reserva, v_numero_habitacion, v_fecha_entrada
    FROM reservaciones 
    WHERE id_reserva = p_id_reserva;
    
    IF v_estado_reserva IS NULL THEN
        SELECT 'Reserva no encontrada' as mensaje;
    ELSEIF v_estado_reserva = 'cancelado' THEN
        SELECT 'Reserva cancelada, no se puede hacer check-in' as mensaje;
    ELSEIF v_fecha_entrada > CURDATE() THEN
        SELECT 'Check-in no permitido antes de la fecha de entrada' as mensaje;
    ELSE
        -- Actualizar estado de reserva y habitación
        UPDATE reservaciones 
        SET estado_reserva = 'check-in', 
            fecha_checkin = NOW(),
            deposito_garantia = p_deposito
        WHERE id_reserva = p_id_reserva;
        
        UPDATE habitaciones SET estado = 'ocupada' WHERE numero_unico = v_numero_habitacion;
        
        SELECT CONCAT('Check-in procesado exitosamente para reserva ', p_id_reserva, '. Depósito: €', p_deposito) as mensaje;
    END IF;
END //

-- 3. RegistrarConsumoServicio
DROP PROCEDURE IF EXISTS RegistrarConsumoServicio//
CREATE PROCEDURE RegistrarConsumoServicio(
    IN p_id_reserva VARCHAR(20),
    IN p_id_servicio INT,
    IN p_cantidad INT,
    IN p_notas TEXT
)
BEGIN
    DECLARE v_costo_servicio DECIMAL(10,2);
    DECLARE v_estado_reserva VARCHAR(20);
    DECLARE v_costo_total DECIMAL(10,2);
    
    -- Verificar estado de reserva
    SELECT estado_reserva INTO v_estado_reserva
    FROM reservaciones 
    WHERE id_reserva = p_id_reserva;
    
    IF v_estado_reserva != 'check-in' THEN
        SELECT 'Servicio solo disponible durante estancia activa (check-in)' as mensaje;
    ELSE
        -- Verificar si el servicio existe
        SELECT costo INTO v_costo_servicio
        FROM servicios_extras 
        WHERE id_servicio = p_id_servicio
        AND disponible = TRUE;
        
        IF v_costo_servicio IS NOT NULL THEN
            SET v_costo_total = v_costo_servicio * p_cantidad;
            
            INSERT INTO reservaciones_servicios (id_reserva, id_servicio, fecha_servicio, hora_servicio, cantidad, costo_unitario, costo_total, notas)
            VALUES (p_id_reserva, p_id_servicio, CURDATE(), CURTIME(), p_cantidad, v_costo_servicio, v_costo_total, p_notas);
            
            SELECT CONCAT('Servicio registrado exitosamente. Costo total: €', v_costo_total) as mensaje;
        ELSE
            SELECT 'Servicio no encontrado o no disponible' as mensaje;
        END IF;
    END IF;
END //

-- 4. ProcesarCheckOut
DROP PROCEDURE IF EXISTS ProcesarCheckOut//
CREATE PROCEDURE ProcesarCheckOut(
    IN p_id_reserva VARCHAR(20)
)
BEGIN
    DECLARE v_total_estancia DECIMAL(10,2);
    DECLARE v_total_servicios DECIMAL(10,2);
    DECLARE v_total_general DECIMAL(10,2);
    DECLARE v_deposito DECIMAL(10,2);
    DECLARE v_dni_huesped VARCHAR(20);
    DECLARE v_numero_habitacion INT;
    DECLARE v_noches INT;
    DECLARE v_puntos INT;
    
    -- Obtener información de la reserva
    SELECT DATEDIFF(fecha_de_salida, fecha_entrada), dni_huesped, numero_habitacion, deposito_garantia, tarifa_aplicada
    INTO v_noches, v_dni_huesped, v_numero_habitacion, v_deposito, v_total_estancia
    FROM reservaciones 
    WHERE id_reserva = p_id_reserva;
    
    SET v_total_estancia = v_total_estancia * v_noches;
    
    -- Calcular servicios adicionales
    SELECT COALESCE(SUM(costo_total), 0) INTO v_total_servicios
    FROM reservaciones_servicios 
    WHERE id_reserva = p_id_reserva;
    
    SET v_total_general = v_total_estancia + v_total_servicios - v_deposito;
    SET v_puntos = FLOOR(v_total_general);
    
    -- Actualizar estado de reserva y habitación
    UPDATE reservaciones 
    SET estado_reserva = 'check-out', 
        estado_pago = IF(v_total_general <= 0, 'pagado', 'pendiente'),
        fecha_checkout = NOW()
    WHERE id_reserva = p_id_reserva;
    
    UPDATE habitaciones SET estado = 'disponible' WHERE numero_unico = v_numero_habitacion;
    
    -- Acumular puntos de fidelización
    UPDATE programa_fidelizacion 
    SET puntos_acumulados = puntos_acumulados + v_puntos,
        total_estancias = total_estancias + 1,
        total_noches = total_noches + v_noches,
        fecha_ultima_actualizacion = CURDATE()
    WHERE dni_huesped = v_dni_huesped;
    
    -- Actualizar nivel según puntos
    UPDATE programa_fidelizacion 
    SET nivel = CASE 
        WHEN puntos_acumulados >= 1000 THEN 'platino'
        WHEN puntos_acumulados >= 500 THEN 'oro'
        WHEN puntos_acumulados >= 200 THEN 'plata'
        ELSE 'bronce'
    END
    WHERE dni_huesped = v_dni_huesped;
    
    SELECT 
        CONCAT('Check-out procesado exitosamente. Total a pagar: €', v_total_general) as mensaje,
        v_total_estancia as costo_estancia,
        v_total_servicios as costo_servicios,
        v_deposito as deposito_aplicado,
        v_total_general as total_final,
        v_puntos as puntos_ganados;
END //

-- 5. ProgramarEvento
DROP PROCEDURE IF EXISTS ProgramarEvento//
CREATE PROCEDURE ProgramarEvento(
    IN p_nombre_evento VARCHAR(100),
    IN p_tipo_evento VARCHAR(50),
    IN p_fecha_evento DATE,
    IN p_hora_inicio TIME,
    IN p_hora_fin TIME,
    IN p_numero_asistentes INT,
    IN p_id_salon INT,
    IN p_id_reserva VARCHAR(20),
    IN p_descripcion TEXT,
    IN p_requerimientos TEXT
)
BEGIN
    DECLARE v_salon_disponible INT;
    DECLARE v_capacidad_salon INT;
    DECLARE v_costo_salon DECIMAL(10,2);
    DECLARE v_duracion INT;
    DECLARE v_costo_total DECIMAL(10,2);
    DECLARE v_id_evento INT;
    
    -- Verificar disponibilidad del salón
    SELECT COUNT(*) INTO v_salon_disponible
    FROM reservas_salones rs
    WHERE rs.id_salon = p_id_salon
    AND rs.fecha = p_fecha_evento
    AND (
        (p_hora_inicio BETWEEN rs.hora_inicio AND rs.hora_fin) OR
        (p_hora_fin BETWEEN rs.hora_inicio AND rs.hora_fin) OR
        (rs.hora_inicio BETWEEN p_hora_inicio AND p_hora_fin)
    )
    AND rs.estado != 'cancelado';
    
    -- Obtener capacidad y costo del salón
    SELECT capacidad, costo_por_hora INTO v_capacidad_salon, v_costo_salon
    FROM salones_eventos 
    WHERE id_salon = p_id_salon;
    
    IF v_salon_disponible > 0 THEN
        SELECT 'Salón no disponible en ese horario' as mensaje;
    ELSEIF p_numero_asistentes > v_capacidad_salon THEN
        SELECT CONCAT('Salón excede capacidad. Máximo: ', v_capacidad_salon, ' asistentes') as mensaje;
    ELSE
        -- Calcular duración y costo
        SET v_duracion = TIMESTAMPDIFF(HOUR, p_hora_inicio, p_hora_fin);
        SET v_costo_total = v_duracion * v_costo_salon;
        
        -- Insertar evento
        INSERT INTO eventos (nombre_evento, tipo_evento, fecha_evento, hora_inicio, hora_fin, numero_asistentes, descripcion, costo_evento, id_reserva, requerimientos_especiales)
        VALUES (p_nombre_evento, p_tipo_evento, p_fecha_evento, p_hora_inicio, p_hora_fin, p_numero_asistentes, p_descripcion, v_costo_total, p_id_reserva, p_requerimientos);
        
        -- Reservar salón
        SET v_id_evento = LAST_INSERT_ID();
        INSERT INTO reservas_salones (id_salon, id_evento, fecha, hora_inicio, hora_fin, costo_total)
        VALUES (p_id_salon, v_id_evento, p_fecha_evento, p_hora_inicio, p_hora_fin, v_costo_total);
        
        -- Actualizar estado del salón
        UPDATE salones_eventos SET estado = 'ocupado' WHERE id_salon = p_id_salon;
        
        SELECT CONCAT('Evento programado exitosamente. ID Evento: ', v_id_evento, '. Costo total: €', v_costo_total) as mensaje;
    END IF;
END //

DELIMITER ;

SELECT '=== PROCEDIMIENTOS ALMACENADOS CREADOS ===' as info;



DELIMITER //

-- 1. Actualizar disponibilidad de habitación
DROP TRIGGER IF EXISTS TR_ActualizarDisponibilidadHabitacion//
CREATE TRIGGER TR_ActualizarDisponibilidadHabitacion
AFTER UPDATE ON reservaciones
FOR EACH ROW
BEGIN
    IF OLD.estado_reserva != NEW.estado_reserva THEN
        IF NEW.estado_reserva = 'check-out' OR NEW.estado_reserva = 'cancelado' THEN
            UPDATE habitaciones SET estado = 'disponible' WHERE numero_unico = NEW.numero_habitacion;
        ELSEIF NEW.estado_reserva = 'check-in' THEN
            UPDATE habitaciones SET estado = 'ocupada' WHERE numero_unico = NEW.numero_habitacion;
        ELSEIF NEW.estado_reserva = 'confirmada' THEN
            UPDATE habitaciones SET estado = 'reservada' WHERE numero_unico = NEW.numero_habitacion;
        END IF;
    END IF;
END //

-- 2. Calcular tarifa según temporada
DROP TRIGGER IF EXISTS TR_CalcularTarifaTemporada//
CREATE TRIGGER TR_CalcularTarifaTemporada
BEFORE INSERT ON reservaciones
FOR EACH ROW
BEGIN
    DECLARE v_multiplicador DECIMAL(3,2);
    DECLARE v_tarifa_base DECIMAL(10,2);
    
    IF NEW.tarifa_aplicada IS NULL AND NEW.numero_habitacion IS NOT NULL THEN
        -- Obtener tarifa base de la habitación
        SELECT tarifa_base INTO v_tarifa_base
        FROM habitaciones 
        WHERE numero_unico = NEW.numero_habitacion;
        
        -- Obtener multiplicador de temporada
        SELECT COALESCE(multiplicador_tarifa, 1.0) INTO v_multiplicador
        FROM temporadas 
        WHERE NEW.fecha_entrada BETWEEN fecha_inicio AND fecha_fin
        LIMIT 1;
        
        SET NEW.tarifa_aplicada = v_tarifa_base * v_multiplicador;
    END IF;
END //

-- 3. Asignar habitación óptima según preferencias
DROP TRIGGER IF EXISTS TR_AsignarHabitacionOptima//
CREATE TRIGGER TR_AsignarHabitacionOptima
BEFORE INSERT ON reservaciones
FOR EACH ROW
BEGIN
    DECLARE v_preferencias TEXT;
    DECLARE v_habitacion_recomendada INT;
    
    IF NEW.numero_habitacion IS NULL THEN
        -- Obtener preferencias del huésped
        SELECT preferencias INTO v_preferencias
        FROM huespedes 
        WHERE dni = NEW.dni_huesped;
        
        -- Lógica básica de asignación basada en preferencias
        IF v_preferencias LIKE '%vista al mar%' THEN
            SELECT numero_unico INTO v_habitacion_recomendada
            FROM habitaciones 
            WHERE caracteristicas_especiales LIKE '%vista al mar%'
            AND estado = 'disponible'
            AND capacidad >= NEW.numero_huespedes
            AND numero_unico NOT IN (
                SELECT numero_habitacion 
                FROM reservaciones 
                WHERE fecha_entrada <= NEW.fecha_de_salida 
                AND fecha_de_salida >= NEW.fecha_entrada
                AND estado_pago != 'cancelado'
            )
            LIMIT 1;
        ELSEIF v_preferencias LIKE '%suite%' OR v_preferencias LIKE '%jacuzzi%' THEN
            SELECT numero_unico INTO v_habitacion_recomendada
            FROM habitaciones 
            WHERE tipo_habitacion = 'Suite'
            AND estado = 'disponible'
            AND capacidad >= NEW.numero_huespedes
            AND numero_unico NOT IN (
                SELECT numero_habitacion 
                FROM reservaciones 
                WHERE fecha_entrada <= NEW.fecha_de_salida 
                AND fecha_de_salida >= NEW.fecha_entrada
                AND estado_pago != 'cancelado'
            )
            LIMIT 1;
        END IF;
        
        -- Si se encontró una habitación recomendada, asignarla
        IF v_habitacion_recomendada IS NOT NULL THEN
            SET NEW.numero_habitacion = v_habitacion_recomendada;
        END IF;
    END IF;
END //

-- 4. Acumular puntos al cliente al hacer check-out
DROP TRIGGER IF EXISTS TR_AcumularPuntosCliente//
CREATE TRIGGER TR_AcumularPuntosCliente
AFTER UPDATE ON reservaciones
FOR EACH ROW
BEGIN
    DECLARE v_total_estancia DECIMAL(10,2);
    DECLARE v_total_servicios DECIMAL(10,2);
    DECLARE v_total_general DECIMAL(10,2);
    DECLARE v_puntos INT;
    DECLARE v_existe INT;
    DECLARE v_noches INT;
    
    IF NEW.estado_reserva = 'check-out' AND OLD.estado_reserva != 'check-out' THEN
        -- Calcular noches de estancia
        SET v_noches = DATEDIFF(NEW.fecha_de_salida, NEW.fecha_entrada);
        SET v_total_estancia = NEW.tarifa_aplicada * v_noches;
        
        -- Calcular servicios adicionales
        SELECT COALESCE(SUM(costo_total), 0) INTO v_total_servicios
        FROM reservaciones_servicios 
        WHERE id_reserva = NEW.id_reserva;
        
        SET v_total_general = v_total_estancia + v_total_servicios;
        SET v_puntos = FLOOR(v_total_general);
        
        -- Verificar si el huésped está en el programa de fidelización
        SELECT COUNT(*) INTO v_existe
        FROM programa_fidelizacion 
        WHERE dni_huesped = NEW.dni_huesped;
        
        IF v_existe = 0 THEN
            -- Insertar nuevo huésped en el programa
            INSERT INTO programa_fidelizacion (dni_huesped, puntos_acumulados, total_estancias, total_noches)
            VALUES (NEW.dni_huesped, v_puntos, 1, v_noches);
        ELSE
            -- Actualizar puntos existentes
            UPDATE programa_fidelizacion 
            SET puntos_acumulados = puntos_acumulados + v_puntos,
                total_estancias = total_estancias + 1,
                total_noches = total_noches + v_noches,
                fecha_ultima_actualizacion = CURDATE()
            WHERE dni_huesped = NEW.dni_huesped;
            
            -- Actualizar nivel según puntos acumulados
            UPDATE programa_fidelizacion 
            SET nivel = CASE 
                WHEN puntos_acumulados >= 1000 THEN 'platino'
                WHEN puntos_acumulados >= 500 THEN 'oro'
                WHEN puntos_acumulados >= 200 THEN 'plata'
                ELSE 'bronce'
            END
            WHERE dni_huesped = NEW.dni_huesped;
        END IF;
    END IF;
END //

-- 5. Verificar ocupación de salones (CORREGIDO)
DROP TRIGGER IF EXISTS TR_VerificarOcupacionSalones//
CREATE TRIGGER TR_VerificarOcupacionSalones
BEFORE INSERT ON reservas_salones
FOR EACH ROW
BEGIN
    DECLARE v_conflictos INT;
    DECLARE v_salon_existe INT;
    
    -- Verificar si el salón existe
    SELECT COUNT(*) INTO v_salon_existe
    FROM salones_eventos 
    WHERE id_salon = NEW.id_salon;
    
    IF v_salon_existe = 0 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'El salón especificado no existe';
    END IF;
    
    -- Verificar conflictos de horario
    SELECT COUNT(*) INTO v_conflictos
    FROM reservas_salones 
    WHERE id_salon = NEW.id_salon
    AND fecha = NEW.fecha
    AND estado != 'cancelado'
    AND (
        (NEW.hora_inicio BETWEEN hora_inicio AND hora_fin) OR
        (NEW.hora_fin BETWEEN hora_inicio AND hora_fin) OR
        (hora_inicio BETWEEN NEW.hora_inicio AND NEW.hora_fin) OR
        (hora_fin BETWEEN NEW.hora_inicio AND NEW.hora_fin)
    );
    
    IF v_conflictos > 0 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Conflicto de horario: el salón ya está reservado en ese horario';
    END IF;
    
    -- Verificar que la hora de fin sea posterior a la hora de inicio
    IF NEW.hora_fin <= NEW.hora_inicio THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'La hora de fin debe ser posterior a la hora de inicio';
    END IF;
END //

DELIMITER ;

SELECT '=== TRIGGERS CREADOS CORRECTAMENTE ===' as info;
