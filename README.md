
-- RESET DATABASE SAFELY

DROP DATABASE IF EXISTS Hospital_managment_system;
CREATE DATABASE Hospital_managment_system;
USE Hospital_managment_system;


-- TABLES (CREATE FRESH EVERY TIME)


CREATE TABLE Admin (
    admin_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    password VARCHAR(50) NOT NULL
);

CREATE TABLE Patient (
    patient_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    age INT CHECK (age > 0),
    gender VARCHAR(10),
    phone VARCHAR(15) UNIQUE,
    address VARCHAR(100)
);

CREATE TABLE Specialization (
    spec_id INT PRIMARY KEY AUTO_INCREMENT,
    spec_name VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE Doctor (
    doctor_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    spec_id INT,
    phone VARCHAR(15) UNIQUE,
    room_no INT UNIQUE,
    fee DECIMAL(10,2) CHECK (fee >= 0),
    FOREIGN KEY (spec_id) REFERENCES Specialization(spec_id)
);

CREATE TABLE Medical_Record (
    record_id INT PRIMARY KEY AUTO_INCREMENT,
    patient_id INT UNIQUE,
    diagnosis TEXT,
    treatment TEXT,
    FOREIGN KEY (patient_id) REFERENCES Patient(patient_id)
);

CREATE TABLE Appointment (
    appointment_id INT PRIMARY KEY AUTO_INCREMENT,
    patient_id INT,
    doctor_id INT,
    appointment_date DATE,
    appointment_time TIME,
    fee DECIMAL(10,2),
    payment_status VARCHAR(20) DEFAULT 'Pending',
    status VARCHAR(20) DEFAULT 'Pending',
    FOREIGN KEY (patient_id) REFERENCES Patient(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES Doctor(doctor_id)
);


-- SAFE INSERT DATA (NO DUPLICATE ERROR)
---------------------------------------------------
-- ADMIN DATA INSERT
---------------------------------------------------

INSERT IGNORE INTO Admin (username, password)
VALUES ('admin_fahim', 'password123'),
       ('hospital_staff', 'secure_pass');
       
INSERT IGNORE INTO Medical_Record (patient_id, diagnosis, treatment)
VALUES (
    (SELECT patient_id FROM Patient WHERE phone = '01992777079'), 
    'Hypertension and Mild Chest Pain', 
    'Prescribed Amlodipine 5mg and 1 week bed rest.'
);

INSERT IGNORE INTO Specialization (spec_name)
VALUES ('Cardiology'), ('Neurology');

INSERT IGNORE INTO Patient (name, age, gender, phone, address)
VALUES ('Kolpona_Rani_Sarker', 51, 'Female', '01992777079', 'Dhaka');

INSERT IGNORE INTO Doctor (name, spec_id, phone, room_no, fee)
SELECT 'Dr. Sandip Kumar Dash', spec_id, '9666-710678', 308, 2500
FROM Specialization
WHERE spec_name = 'Cardiology';


-- SAFE APPOINTMENT INSERT


INSERT INTO Appointment(patient_id, doctor_id, appointment_date, appointment_time, fee)
SELECT 
    (SELECT patient_id FROM Patient WHERE phone = '01992777079'),
    (SELECT doctor_id FROM Doctor WHERE phone = '9666-710678'),
    '2026-04-23',
    '10:00:00',
    (SELECT fee FROM Doctor WHERE phone = '9666-710678');


-- UPDATE SAFE (ONLY IF EXISTS)


UPDATE Appointment
SET payment_status = 'Paid'
WHERE appointment_id = 1;

UPDATE Appointment
SET status = 'Completed'
WHERE appointment_id = 1;


-- JOIN QUERY


SELECT 
    P.name AS Patient,
    D.name AS Doctor,
    A.fee,
    A.payment_status,
    A.status
FROM Appointment A
JOIN Patient P ON A.patient_id = P.patient_id
JOIN Doctor D ON A.doctor_id = D.doctor_id;


-- FUNCTION (SAFE)

DROP FUNCTION IF EXISTS TotalAppointments;

DELIMITER //

CREATE FUNCTION TotalAppointments(pid INT)
RETURNS INT
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE total_count INT;
    
    SELECT COUNT(*) INTO total_count 
    FROM Appointment 
    WHERE patient_id = pid;
    
    RETURN total_count;
END //

DELIMITER ;


-- PROCEDURE (SAFE DROP + CREATE STYLE)


DROP PROCEDURE IF EXISTS BookAppointment;

DELIMITER //

CREATE PROCEDURE BookAppointment(
    IN p_phone VARCHAR(15),
    IN d_phone VARCHAR(15),
    IN app_date DATE,
    IN app_time TIME
)
BEGIN
    INSERT INTO Appointment(patient_id, doctor_id, appointment_date, appointment_time, fee)
    SELECT 
        (SELECT patient_id FROM Patient WHERE phone = p_phone),
        (SELECT doctor_id FROM Doctor WHERE phone = d_phone),
        app_date,
        app_time,
        (SELECT fee FROM Doctor WHERE phone = d_phone);
END //

DELIMITER ;


-- TRIGGER (SAFE RECREATE)


DROP TRIGGER IF EXISTS auto_payment_status;

DELIMITER //

CREATE TRIGGER auto_payment_status
BEFORE UPDATE ON Appointment
FOR EACH ROW
BEGIN
    IF NEW.status = 'Completed' THEN
        SET NEW.payment_status = 'Paid';
    END IF;
END //

DELIMITER ;


-- CURSOR PROCEDURE (SAFE)


DROP PROCEDURE IF EXISTS ShowPatients;

DELIMITER //

CREATE PROCEDURE ShowPatients()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE pname VARCHAR(50);

    DECLARE cur CURSOR FOR SELECT name FROM Patient;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    OPEN cur;

    read_loop: LOOP
        FETCH cur INTO pname;
        IF done THEN
            LEAVE read_loop;
        END IF;
        SELECT pname;
    END LOOP;

    CLOSE cur;
END //

DELIMITER ;


SELECT * FROM Admin;
SELECT * FROM Patient;
SELECT * FROM Specialization;
SELECT * FROM Doctor;
SELECT * FROM Medical_Record;
SELECT * FROM Appointment;
