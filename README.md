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
