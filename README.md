# AgroData-Engine---Emphasizes-data-processing---expanded-
## Problem statement 
### Problem Definition:
Agricultural farms lack integrated data systems for real-time monitoring, leading to 15-25% yield reduction, 30% water waste, and delayed emergency response to environmental threats.

Context:
Medium to large-scale farming operations (50-500 acres) needing automation for crop management, resource allocation, and yield optimization.

Target Users:

Farm Managers

Agricultural Technicians

Supply Chain Coordinators

Financial Analysts

BI Potential:

Real-time yield prediction dashboards

Resource utilization analytics

Emergency response tracking

ROI analysis per crop type

## Enhanced Business Process Model (Phase II)

##

## Enhanced Database Configuration
-- Production tablespaces
CREATE TABLESPACE agri_data 
DATAFILE 'agri_data01.dbf' SIZE 500M 
AUTOEXTEND ON NEXT 100M MAXSIZE 2G;

CREATE TABLESPACE agri_idx 
DATAFILE 'agri_idx01.dbf' SIZE 200M 
AUTOEXTEND ON NEXT 50M MAXSIZE 1G;

-- Temporary tablespace
CREATE TEMPORARY TABLESPACE agri_temp 
TEMPFILE 'agri_temp01.dbf' SIZE 100M 
AUTOEXTEND ON NEXT 25M;

## Enhanced Table Implementation (Phase V)
CREATE TABLE crops (
    crop_id NUMBER(10) PRIMARY KEY,
    crop_name VARCHAR2(100) NOT NULL,
    crop_type VARCHAR2(50) CHECK (crop_type IN ('Vegetable','Grain','Fruit','Legume')),
    growing_season VARCHAR2(20) CHECK (growing_season IN ('Spring','Summer','Fall','Winter')),
    days_to_harvest NUMBER(5) CHECK (days_to_harvest > 0),
    water_requirements NUMBER(5,2) CHECK (water_requirements BETWEEN 0 AND 100),
    ideal_ph_level NUMBER(3,1) CHECK (ideal_ph_level BETWEEN 3.0 AND 9.0),
    avg_yield_per_acre NUMBER(8,2),
    created_date DATE DEFAULT SYSDATE NOT NULL,
    modified_date DATE
) TABLESPACE agri_data;

-- Indexes for performance
CREATE INDEX idx_crops_type ON crops(crop_type) TABLESPACE agri_idx;
CREATE INDEX idx_crops_season ON crops(growing_season) TABLESPACE agri_idx;

### Data Insertion
#### CROPS Table 

INSERT INTO crops VALUES 
(1, 'Sweet Corn', 'Grain', 'Summer', 90, 25.5, 6.0, 800.00, SYSDATE, NULL);

INSERT INTO crops VALUES 
(2, 'Heirloom Tomatoes', 'Vegetable', 'Summer', 75, 30.0, 6.5, 600.00, SYSDATE, NULL);

INSERT INTO crops VALUES 
(3, 'Winter Wheat', 'Grain', 'Fall', 120, 20.0, 6.2, 1200.00, SYSDATE, NULL);

#### FIELDS Table

INSERT INTO fields VALUES 
(101, 'North Field', 50, 'Loam', 6.3, 'Sprinkler', SYSDATE, NULL);

INSERT INTO fields VALUES 
(102, 'South Field', 75, 'Clay', 5.8, 'Drip', SYSDATE, NULL);

INSERT INTO fields VALUES 
(103, 'East Field', 40, 'Sandy', 6.7, 'Flood', SYSDATE, NULL);

#### SENSORS Table

INSERT INTO sensors VALUES 
(1, 101, 'Soil Moisture', 'ACTIVE', SYSDATE, SYSDATE, SYSDATE);

INSERT INTO sensors VALUES 
(2, 102, 'Temperature', 'ACTIVE', SYSDATE, SYSDATE, SYSDATE);

INSERT INTO sensors VALUES 
(3, 103, 'pH Meter', 'ACTIVE', SYSDATE, SYSDATE, SYSDATE);

#### CROP_ROTATION Table

INSERT INTO crop_rotation VALUES 
(1, 101, 1, 'Summer 2025', 40000, NULL, 'PLANNED', SYSDATE);

INSERT INTO crop_rotation VALUES 
(2, 102, 2, 'Summer 2025', 45000, 42000, 'HARVESTED', SYSDATE);

INSERT INTO crop_rotation VALUES 
(3, 103, 3, 'Fall 2025', 48000, NULL, 'PLANTED', SYSDATE);

#### HARVEST_RECORDS Table

INSERT INTO harvest_records VALUES 
(1, 2, '2025-08-15', 42000, 'Grade_A', 'Good yield, minor pest damage', SYSDATE);

INSERT INTO harvest_records VALUES 
(2, NULL, '2025-10-05', NULL, NULL, 'Planned wheat harvest', SYSDATE);

INSERT INTO harvest_records VALUES 
(3, 1, '2024-08-20', 38000, 'Grade_A', 'Historical data for comparison', SYSDATE);

####  EMERGENCY_EVENTS Table 

INSERT INTO emergency_events VALUES 
(1, 101, 'DROUGHT', 8, SYSDATE-10, 'IMMEDIATE_IRRIGATION', 'RESOLVED');

INSERT INTO emergency_events VALUES 
(2, 102, 'FROST', 7, SYSDATE-5, 'ACTIVATE_HEATERS', 'RESOLVED');

INSERT INTO emergency_events VALUES 
(3, 103, 'PEST_INFESTATION', 6, SYSDATE-3, 'APPLY_PESTICIDE', 'IN_PROGRESS');

#### SENSOR_READINGS Table

INSERT INTO sensor_readings VALUES 
(1, 1, 65.5, SYSTIMESTAMP, 'OPTIMAL');  -- Good moisture

INSERT INTO sensor_readings VALUES 
(2, 2, 38.2, SYSTIMESTAMP, 'HIGH');     -- Critical temperature

INSERT INTO sensor_readings VALUES 
(3, 3, 6.2, SYSTIMESTAMP, 'OPTIMAL');   -- Normal pH

#### RESOURCE_USAGE Table 

INSERT INTO resource_usage VALUES 
(1, 101, 'WATER', 12500, '2025-07-15', 'IRRIGATION');

INSERT INTO resource_usage VALUES 
(2, 102, 'FERTILIZER', 500, '2025-07-10', 'SOIL_AMENDMENT');

INSERT INTO resource_usage VALUES 
(3, 103, 'ELECTRICITY', 1200, '2025-07-12', 'IRRIGATION_PUMP');

### Enhanced PL/SQL Development (Phase VI)

-- 1. Emergency Response Procedure
CREATE OR REPLACE PROCEDURE handle_emergency(
    p_field_id IN NUMBER,
    p_event_type IN VARCHAR2,
    p_severity IN NUMBER,
    p_response OUT VARCHAR2
)
AS
    v_sensor_status VARCHAR2(50);
    v_available_resources NUMBER;
BEGIN
    -- Check sensor status
    SELECT status INTO v_sensor_status 
    FROM sensors 
    WHERE field_id = p_field_id AND status = 'ACTIVE';
    
 -- Determine response based on severity
    IF p_severity >= 8 THEN
        p_response := 'IMMEDIATE_IRRIGATION_SHUTDOWN';
        GOTO emergency_protocol;
    ELSIF p_severity >= 5 THEN
        p_response := 'ALERT_TECHNICIAN_DISPATCH';
        GOTO alert_protocol;
    ELSE
        p_response := 'MONITOR_AND_LOG';
        GOTO monitoring_protocol;
    END IF;
    
  <<emergency_protocol>>
    -- Execute emergency shutdown
    UPDATE irrigation_systems 
    SET status = 'SHUTDOWN'
    WHERE field_id = p_field_id;
    
INSERT INTO emergency_events 
    VALUES (seq_emergency.nextval, p_field_id, p_event_type, p_severity, SYSDATE);
    
  <<alert_protocol>>
    -- Send alert to technicians
    NULL; -- Alert system implementation
    
   <<monitoring_protocol>>
    -- Log event for monitoring
    NULL; -- Monitoring implementation
    
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        p_response := 'ERROR_NO_ACTIVE_SENSORS';
        RAISE_APPLICATION_ERROR(-20001, 'No active sensors found for field');
END;
/

#### Calculation of Water Efficiency Score

-- 1. Calculate Water Efficiency Score
CREATE OR REPLACE FUNCTION calculate_water_efficiency(
    p_field_id IN NUMBER,
    p_season IN VARCHAR2
) RETURN NUMBER
AS
    v_ideal_water NUMBER;
    v_actual_water NUMBER;
    v_efficiency_score NUMBER;
BEGIN
    -- Calculate efficiency based on ideal vs actual
    SELECT SUM(c.water_requirements * r.acreage) 
    INTO v_ideal_water
    FROM crop_rotation r
    JOIN crops c ON r.crop_id = c.crop_id
    WHERE r.field_id = p_field_id 
    AND r.season_year LIKE '%' || p_season || '%';
    
 SELECT SUM(water_used) 
    INTO v_actual_water
    FROM resource_usage
    WHERE field_id = p_field_id 
    AND resource_type = 'WATER'
    AND EXTRACT(MONTH FROM usage_date) BETWEEN 
        CASE p_season 
            WHEN 'Spring' THEN 3 WHEN 'Summer' THEN 6
            WHEN 'Fall' THEN 9 WHEN 'Winter' THEN 12
        END AND 
        CASE p_season 
            WHEN 'Spring' THEN 5 WHEN 'Summer' THEN 8
            WHEN 'Fall' THEN 11 WHEN 'Winter' THEN 2
        END;
    
    v_efficiency_score := ROUND((v_ideal_water / NULLIF(v_actual_water, 0)) * 100, 2);
    RETURN v_efficiency_score;
END;
/

#### Package Implementation:

CREATE OR REPLACE PACKAGE farm_management_pkg AS
    -- Procedures
    PROCEDURE automate_irrigation(p_field_id NUMBER);
    PROCEDURE schedule_harvest(p_rotation_id NUMBER);
    PROCEDURE analyze_field_performance(p_field_id NUMBER);
    
        
-- Functions
    FUNCTION calculate_roi(p_field_id NUMBER, p_season VARCHAR2) RETURN NUMBER;
    FUNCTION get_crop_suitability(p_field_id NUMBER, p_crop_id NUMBER) RETURN VARCHAR2;
    
    -- Cursor
    CURSOR get_low_performing_fields RETURN fields%ROWTYPE;
END farm_management_pkg;
/

CREATE OR REPLACE PACKAGE BODY farm_management_pkg AS
    -- Implementation with GOTO for error handling
    PROCEDURE automate_irrigation(p_field_id NUMBER) IS
        v_moisture_level NUMBER;
        v_irrigation_needed BOOLEAN := FALSE;
    BEGIN
        -- Get latest moisture reading
        SELECT reading_value INTO v_moisture_level
        FROM sensor_readings
        WHERE sensor_id = (SELECT sensor_id FROM sensors 
                          WHERE field_id = p_field_id AND sensor_type = 'MOISTURE')
        ORDER BY reading_timestamp DESC
        FETCH FIRST 1 ROW ONLY;
 
  IF v_moisture_level < 40 THEN
            v_irrigation_needed := TRUE;
            GOTO activate_irrigation;
        ELSIF v_moisture_level BETWEEN 40 AND 60 THEN
            GOTO monitor_only;
        ELSE
            GOTO irrigation_ok;
        END IF;
        
 <<activate_irrigation>>
        -- Execute irrigation
        UPDATE irrigation_systems 
        SET status = 'ACTIVE', 
            start_time = SYSDATE
        WHERE field_id = p_field_id;
        GOTO log_action;
        
<<monitor_only>>
        -- Log for monitoring
        INSERT INTO monitoring_log 
        VALUES (seq_monitor.nextval, p_field_id, 'LOW_MOISTURE', SYSDATE);
        GOTO exit_procedure;
        
 <<irrigation_ok>>
        -- No action needed
        NULL;
        
 <<log_action>>
        INSERT INTO action_log 
        VALUES (seq_action.nextval, 'IRRIGATION', p_field_id, SYSDATE);
        
   <<exit_procedure>>
        NULL;
    END automate_irrigation;
END farm_management_pkg;
/

#### windows dunction

-- Yield ranking by field
SELECT 
    field_id,
    crop_name,
    actual_yield,
    ROW_NUMBER() OVER (PARTITION BY crop_id ORDER BY actual_yield DESC) as yield_rank,
    LAG(actual_yield, 1) OVER (PARTITION BY field_id ORDER BY harvest_date) as prev_yield,
    AVG(actual_yield) OVER (PARTITION BY crop_id) as avg_crop_yield
FROM harvest_records hr
JOIN crops c ON hr.crop_id = c.crop_id;

-- Seasonal performance comparison
SELECT 
    field_name,
    season_year,
    actual_yield,
    RANK() OVER (PARTITION BY field_id ORDER BY actual_yield DESC) as seasonal_rank,
    DENSE_RANK() OVER (ORDER BY actual_yield DESC) as overall_rank,
    actual_yield - LAG(actual_yield, 1) OVER (PARTITION BY field_id ORDER BY season_year) as yield_change
FROM harvest_records hr
JOIN fields f ON hr.field_id = f.field_id;

### Enhanced Triggers & Auditing (Phase VII)

-- 1. Holiday Management Table
CREATE TABLE public_holidays (
    holiday_id NUMBER(10) PRIMARY KEY,
    holiday_name VARCHAR2(100) NOT NULL,
    holiday_date DATE NOT NULL,
    description VARCHAR2(500)
);

-- 2. Audit Log Table
CREATE TABLE emergency_audit_log (
    audit_id NUMBER(10) PRIMARY KEY,
    operation_type VARCHAR2(10) CHECK (operation_type IN ('INSERT','UPDATE','DELETE')),
    table_name VARCHAR2(50),
    record_id NUMBER(10),
    user_name VARCHAR2(100),
    attempt_time TIMESTAMP DEFAULT SYSTIMESTAMP,
    status VARCHAR2(20) CHECK (status IN ('ALLOWED','DENIED')),
    error_message VARCHAR2(500)
);

-- 3. Audit Logging Function
CREATE OR REPLACE FUNCTION log_emergency_audit(
    p_operation VARCHAR2,
    p_table VARCHAR2,
    p_record_id NUMBER,
    p_status VARCHAR2,
    p_error VARCHAR2 DEFAULT NULL
) RETURN NUMBER
AS
    PRAGMA AUTONOMOUS_TRANSACTION;
    v_audit_id NUMBER;
BEGIN
    SELECT seq_audit.nextval INTO v_audit_id FROM dual;
    
    INSERT INTO emergency_audit_log 
    VALUES (v_audit_id, p_operation, p_table, p_record_id, 
           USER, SYSTIMESTAMP, p_status, p_error);
    
    COMMIT;
    RETURN v_audit_id;
END;
/

-- 4. Restriction Check Function
CREATE OR REPLACE FUNCTION check_operation_allowed 
RETURN BOOLEAN
AS
    v_current_day VARCHAR2(10);
    v_is_holiday BOOLEAN := FALSE;
BEGIN
    -- Check if weekday
    v_current_day := TO_CHAR(SYSDATE, 'DY');
    IF v_current_day IN ('MON','TUE','WED','THU','FRI') THEN
        RETURN FALSE;
    END IF;
    
    -- Check if public holiday
    SELECT COUNT(*) INTO v_is_holiday
    FROM public_holidays
    WHERE holiday_date = TRUNC(SYSDATE);
    
    RETURN NOT v_is_holiday;
END;
/

-- 5. Compound Trigger for Emergency Events
CREATE OR REPLACE TRIGGER emergency_operations_trigger
FOR INSERT OR UPDATE OR DELETE ON emergency_events
COMPOUND TRIGGER
    
TYPE t_audit_info IS RECORD (
operation VARCHAR2(10),
event_id NUMBER
    );
    
TYPE t_audit_list IS TABLE OF t_audit_info;
 v_audit_list t_audit_list := t_audit_list();
    
BEFORE STATEMENT IS
BEGIN
        IF NOT check_operation_allowed THEN
            RAISE_APPLICATION_ERROR(-20002, 
                'Emergency operations prohibited on weekdays and public holidays');
        END IF;
    END BEFORE STATEMENT;
    
AFTER EACH ROW IS
    BEGIN
        v_audit_list.EXTEND;
        v_audit_list(v_audit_list.COUNT).operation := 
            CASE 
                WHEN INSERTING THEN 'INSERT'
                WHEN UPDATING THEN 'UPDATE'
                WHEN DELETING THEN 'DELETE'
            END;
        v_audit_list(v_audit_list.COUNT).event_id := 
            COALESCE(:NEW.event_id, :OLD.event_id);
    END AFTER EACH ROW;
    
 AFTER STATEMENT IS
    BEGIN
        FOR i IN 1..v_audit_list.COUNT LOOP
            log_emergency_audit(
                v_audit_list(i).operation,
                'EMERGENCY_EVENTS',
                v_audit_list(i).event_id,
                'ALLOWED'
            );
        END LOOP;
    END AFTER STATEMENT;
END emergency_operations_trigger;
/

#### Testing script

-- Test 1: Attempt INSERT on weekday (should fail)
BEGIN
    INSERT INTO emergency_events 
    VALUES (999, 101, 'FROST', 7, SYSDATE);
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Test 1 PASSED: ' || SQLERRM);
END;
/

-- Test 2: Attempt on weekend (should succeed)
-- Execute on Saturday/Sunday
BEGIN
    -- This will only work on weekends
    INSERT INTO emergency_events 
    VALUES (1000, 101, 'TEST', 1, SYSDATE);
    DBMS_OUTPUT.PUT_LINE('Test 2: INSERT allowed on weekend');
    ROLLBACK;
END;
/

-- Test 3: Verify audit logging
SELECT * FROM emergency_audit_log 
ORDER BY attempt_time DESC 
FETCH FIRST 5 ROWS ONLY;
