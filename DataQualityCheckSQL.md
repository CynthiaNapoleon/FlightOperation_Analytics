``` sql query 
Use Flight_Analytics;

select * from  [flight_ops];
select * from [crew_scheduling ];

UPDATE dbo.[flight_ops]
SET Gate_id = 'SEA-G3'
WHERE Origin_airport = 'SEA' 
  AND (Gate_id IS NULL OR Gate_id = '' OR Gate_id = ' ');

UPDATE dbo.[crew_scheduling ]
SET Crew_role = LTRIM(RTRIM(Crew_role));

--Fills NULLs/0s with the average for that specific role
--MEAN IMPUTATION (TOTAL DUTY HOURS)
WITH RoleAverages AS (
    SELECT Crew_role, AVG(Total_duty_hours) AS AvgDutyHours
    FROM [crew_scheduling ]
    WHERE Total_duty_hours IS NOT NULL AND Total_duty_hours > 0
    GROUP BY Crew_role
)
UPDATE cs
SET cs.Total_duty_hours = ra.AvgDutyHours
FROM [crew_scheduling ] cs
JOIN RoleAverages ra ON cs.Crew_role = ra.Crew_role
WHERE cs.Total_duty_hours IS NULL OR cs.Total_duty_hours = 0;

--BUSINESS LOGIC ALIGNMENT (STATUS VS TIMES)
-- Ensures Cancelled flights do not have arrival times (Validity Check)
UPDATE [flight_ops]
SET Actual_departure_time = NULL
WHERE Flight_status = 'Cancelled';

SELECT DISTINCT fo.Crew_id AS Orphaned_Crew_ID
FROM [flight_ops] fo
LEFT JOIN [crew_scheduling ] cs ON fo.Crew_id = cs.Crew_id
WHERE cs.Crew_id IS NULL AND fo.Crew_id <> 'NO_CREW';

SELECT Flight_id, Scheduled_departure_date, Actual_departure_date
FROM [flight_ops]
WHERE Actual_departure_date < Scheduled_departure_date 
  AND Flight_status <> 'Cancelled';

-- Check for NULLs or empty strings in key operational columns
SELECT 
    'flightOps' as TableName,
    SUM(CASE WHEN Flight_id IS NULL OR Flight_id = '' THEN 1 ELSE 0 END) as Null_FlightIDs,
    SUM(CASE WHEN Origin_airport IS NULL THEN 1 ELSE 0 END) as Null_Origins,
    SUM(CASE WHEN Delay_in_departure_min IS NULL AND Flight_status<>'Cancelled' THEN 1 ELSE 0 END) as Null_Delays
FROM flight_ops

UNION ALL

SELECT 
    'CrewScheduling',
    SUM(CASE WHEN Crew_id IS NULL THEN 1 ELSE 0 END),
    SUM(CASE WHEN Crew_base_airport IS NULL THEN 1 ELSE 0 END),
    SUM(CASE WHEN Total_duty_hours IS NULL THEN 1 ELSE 0 END)
FROM [crew_scheduling ]

UNION ALL

SELECT 
    'CrewIncidents',
    SUM(CASE WHEN Incident_id IS NULL THEN 1 ELSE 0 END),
    SUM(CASE WHEN Crew_id IS NULL THEN 1 ELSE 0 END),
    SUM(CASE WHEN Severity_level IS NULL THEN 1 ELSE 0 END)
FROM crew_incident;

--Integrity check
-- Find Incidents linked to a Crew_id that does not exist in the Scheduling file
SELECT DISTINCT i.Crew_id
FROM crew_incident i
LEFT JOIN [crew_scheduling ] s ON i.Crew_id = s.Crew_id
WHERE s.Crew_id IS NULL;

-- Find Flights where the Tail Number is missing (prevents Aircraft Health analysis)
SELECT COUNT(*) as MissingTailCount
FROM flight_ops
WHERE Aircraft_tail_number IS NULL OR Aircraft_tail_number = '';

--Uniqueness 
-- Identify duplicate Primary Keys
SELECT Flight_id, COUNT(*) as DuplicateCount
FROM flight_ops
GROUP BY Flight_id
HAVING COUNT(*) > 1;

SELECT Crew_duty_id, COUNT(*) as DuplicateCount
FROM [crew_scheduling ]
GROUP BY Crew_duty_id
HAVING COUNT(*) > 1;

SELECT Incident_id, COUNT(*) as DuplicateCount
FROM crew_incident
GROUP BY Incident_id
HAVING COUNT(*) > 1;


--Reasonableness check 
-- Check for unrealistic duty or delay values
SELECT 'High Duty Hours (>20h)' as Issue, COUNT(*) as Count FROM [crew_scheduling ] WHERE Total_duty_hours > 20
UNION ALL
SELECT 'Negative Delays' as Issue, COUNT(*) FROM flight_ops WHERE Delay_in_departure_min < 0
UNION ALL
SELECT 'Zero/Negative Rest' as Issue, COUNT(*) FROM [crew_scheduling ] WHERE Layover_duration_hours <= 0;


--Cleaning for duty hours and negative rest 
-- 1. Cap extreme duty hours to 20 to preserve the record but limit skew
UPDATE [crew_scheduling ]
SET Total_duty_hours = 14.5
WHERE Total_duty_hours > 20;

-- 2. Handle Zero Rest by assigning the minimum legal rest (usually 10 hours) 
-- so our ratios don't break
UPDATE [crew_scheduling ]
SET Layover_duration_hours = 10
WHERE Layover_duration_hours <= 0;


```
