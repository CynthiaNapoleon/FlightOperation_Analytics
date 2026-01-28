-- # Area 1: Network Operations (Network Congestion Audit)
```sql
-- a)	What are the primary bottlenecks at the airports of our network?
CREATE VIEW vw_TopDelayReasons AS
WITH DelayCounts AS (
    SELECT 
        Origin_airport,
        Specific_delay_reason,
        COUNT(*) as Total_Occurrences,
        AVG(CAST(Delay_in_departure_min AS FLOAT)) as Avg_Delay_Time,
        RANK() OVER (PARTITION BY Origin_airport ORDER BY COUNT(*) DESC) as Reason_Rank
    FROM flight_ops
    WHERE Specific_delay_reason != 'UNKNOWN'
    GROUP BY Origin_airport, Specific_delay_reason
)
SELECT 
    Origin_airport,
    Specific_delay_reason,
    Total_Occurrences,
    ROUND(Avg_Delay_Time, 2) as Avg_Delay_Minutes
FROM DelayCounts
WHERE Reason_Rank <= 3;


--  b)	Which aircraft are dragging down the fleet reliability?
CREATE VIEW vw_AircraftReliability AS
SELECT 
    Aircraft_tail_number,
    -- Truncates date to the first of the month 
    DATEFROMPARTS(YEAR(CONVERT(DATE, scheduled_departure_date, 105)), MONTH(CONVERT(DATE, scheduled_departure_date, 105)), 1) as Flight_Month,
    COUNT(Flight_id) as Total_Flights,
    SUM(CASE WHEN Flight_status = 'CANCELLED' THEN 1 ELSE 0 END) as Total_Cancellations,
    ROUND(AVG(CAST(Delay_in_departure_min AS FLOAT)), 2) as Avg_Delay_Minutes,
    MAX(Delay_in_departure_min) as Max_Single_Delay
FROM flight_ops
GROUP BY Aircraft_tail_number, 
         DATEFROMPARTS(YEAR(CONVERT(DATE, scheduled_departure_date, 105)), MONTH(CONVERT(DATE, scheduled_departure_date, 105)), 1);


-- c) Where is the utilization inefficiency draining our flight capacity?	
CREATE VIEW vw_UtilizationInefficiency AS
SELECT 
    Origin_airport,
    ROUND(AVG(CAST(Turnaround_time_min AS FLOAT)), 2) as Actual_Avg_Turnaround,
    ROUND(AVG(CAST(Scheduled_block_time_min AS FLOAT)), 2) as Avg_Scheduled_Block,
    ROUND(AVG(CAST(Turnaround_time_min - Scheduled_block_time_min AS FLOAT)), 2) as Utilization_Inefficiency
FROM flight_ops
WHERE Flight_status = 'DEPARTED'
GROUP BY Origin_airport
HAVING COUNT(Flight_id) > 100;


-- d)	When is our network most vulnerable to failure? 
CREATE VIEW vw_PeakHourAnalysis AS
SELECT 
    Origin_airport,
    DATEPART(HOUR, CAST(scheduled_departure_time AS TIME)) as Departure_Hour,
    COUNT(Flight_id) as Flight_Volume,
    ROUND(AVG(CAST(Delay_in_departure_min AS FLOAT)), 2) as Avg_Delay
FROM flight_ops
GROUP BY Origin_airport, DATEPART(HOUR, CAST(scheduled_departure_time AS TIME));


-- e)	Which station is best at catching up the schedule? 
CREATE VIEW vw_TimeRecovery AS
SELECT 
    Flight_id,
    Origin_airport,
    Delay_in_departure_min as Departure_Delay,
    Turnaround_time_min,
    CASE 
        WHEN Turnaround_time_min < (SELECT AVG(f2.Turnaround_time_min) FROM flight_ops f2 WHERE f2.Origin_airport = f1.Origin_airport)
        THEN 'Recovered Time' 
        ELSE 'Lost Time' 
    END as Recovery_Status
FROM flight_ops f1
WHERE Delay_in_departure_min > 15;


-- f) Which routes are the most unpredictable?
CREATE VIEW vw_RouteReliability AS
SELECT 
    Origin_airport,
    Destination_airport,
    COUNT(Flight_id) as Flight_Count,
    ROUND(AVG(CAST(Delay_in_departure_min AS FLOAT)), 2) as Avg_Delay,
    ROUND(STDEV(Delay_in_departure_min), 2) as Delay_Standard_Deviation
FROM flight_ops
GROUP BY Origin_airport, Destination_airport
HAVING COUNT(Flight_id) > 50;

```



-- # Area 2: Crew Logistics (Workforce Constraints)
```sql
--a) Is our base network operating at a consistent efficiency? 
--Calculates how each crew base performs compared to the corporate average.
CREATE VIEW vw_BaseEfficiency AS
WITH BaseStats AS (
    SELECT 
        Crew_base_airport,
        AVG(CAST(Total_duty_hours AS FLOAT)) as Avg_Base_Duty,
        -- Converts HH:MM:SS to total minutes in T-SQL
        AVG(CAST(DATEPART(HOUR, CAST(Delay_in_starting AS TIME)) * 60 + 
            DATEPART(MINUTE, CAST(Delay_in_starting AS TIME)) AS FLOAT)) as Avg_Base_Start_Delay
    FROM crew_scheduling
    GROUP BY Crew_base_airport
)
SELECT 
    Crew_base_airport,
    ROUND(Avg_Base_Duty, 2) as Avg_Base_Duty,
    ROUND(Avg_Base_Start_Delay, 2) as Avg_Base_Start_Delay,
    ROUND(AVG(Avg_Base_Duty) OVER(), 2) as Company_Avg_Duty,
    ROUND(Avg_Base_Duty - AVG(Avg_Base_Duty) OVER(), 2) as Duty_Variance
FROM BaseStats;

--b) Which professional groups face the most friction starting their day? 
--Identifies which crew roles (Pilots, Cabin Crew) face the most friction at sign-in.
CREATE VIEW vw_RoleDelayImpact AS
SELECT 
    Crew_role,
    COUNT(*) as Duty_Count,
    ROUND(AVG(CAST(DATEPART(HOUR, CAST(Delay_in_starting AS TIME)) * 60 + 
              DATEPART(MINUTE, CAST(Delay_in_starting AS TIME)) AS FLOAT)), 2) as Avg_Start_Delay_Min
FROM crew_scheduling
GROUP BY Crew_role;

--c) Where are we paying for "Idle Time" instead of "Flying Time"? (Layover Optimization - Instances > 24 hours)
--Highlights bases where crew are sitting idle for too long.
CREATE VIEW vw_LayoverOptimization AS
SELECT 
    Crew_base_airport,
    ROUND(AVG(CAST(Layover_duration_hours AS FLOAT)), 2) as Avg_Layover_Hrs,
    MAX(Layover_duration_hours) as Max_Layover_Hrs,
    SUM(CASE WHEN Layover_duration_hours > 24 THEN 1 ELSE 0 END) as Layovers_Over_24h
FROM crew_scheduling
GROUP BY Crew_base_airport;

--d) Is the workload distributed fairly across our people? (Workload Equity - StdDev of Duty Hours).
--Uses STDEV to see if some crew members are overworked while others are underutilized.
CREATE VIEW vw_WorkloadEquity AS
SELECT 
    Crew_base_airport,
    Crew_role,
    ROUND(STDEV(Total_duty_hours), 2) as Duty_Hour_StdDev,
    ROUND(MAX(Total_duty_hours) - MIN(Total_duty_hours), 2) as Workload_Range_Hrs,
    COUNT(DISTINCT Crew_id) as Crew_Count
FROM crew_scheduling
GROUP BY Crew_base_airport, Crew_role
HAVING COUNT(*) > 5;

--e) Is our "Morning Rush" the cause of our daily delays? (Duty Start Clusters / Shift Analysis).
--Groups shifts into categories to find "problem times."
CREATE VIEW vw_DutyStartClusters AS
SELECT 
    CASE 
        WHEN DATEPART(HOUR, CAST(Scheduled_start_time AS TIME)) BETWEEN 4 AND 8 THEN 'Early Morning Rush'
        WHEN DATEPART(HOUR, CAST(Scheduled_start_time AS TIME)) BETWEEN 9 AND 17 THEN 'Daytime Shift'
        WHEN DATEPART(HOUR, CAST(Scheduled_start_time AS TIME)) BETWEEN 18 AND 22 THEN 'Evening Shift'
        ELSE 'Night Shift'
    END as Shift_Category,
    ROUND(AVG(CAST(DATEPART(HOUR, CAST(Delay_in_starting AS TIME)) * 60 + 
              DATEPART(MINUTE, CAST(Delay_in_starting AS TIME)) AS FLOAT)), 2) as Avg_Start_Delay_Min,
    COUNT(*) as Shift_Count
FROM crew_scheduling
GROUP BY 
    CASE 
        WHEN DATEPART(HOUR, CAST(Scheduled_start_time AS TIME)) BETWEEN 4 AND 8 THEN 'Early Morning Rush'
        WHEN DATEPART(HOUR, CAST(Scheduled_start_time AS TIME)) BETWEEN 9 AND 17 THEN 'Daytime Shift'
        WHEN DATEPART(HOUR, CAST(Scheduled_start_time AS TIME)) BETWEEN 18 AND 22 THEN 'Evening Shift'
        ELSE 'Night Shift'
    END;

```

-- # Area 3: Safety & Vulnerability (Systemic Fragility)
```sql
-- a) Is our safety risk being driven by crew exhaustion from previous workloads? (Correlation of 7-day rolling duty hours against Fatigue_Risk_Category)
--Identifies if incidents are linked to high workloads in the week prior.
CREATE VIEW vw_FatigueCorrelation AS
SELECT 
    i.Incident_id,
    i.Crew_id,
    i.Incident_date,
    i.Severity_level,
    SUM(s.Total_duty_hours) as Duty_Hours_Last_7_Days,
    CASE 
        WHEN SUM(s.Total_duty_hours) > 50 THEN 'High Risk'
        WHEN SUM(s.Total_duty_hours) BETWEEN 35 AND 50 THEN 'Moderate Risk'
        ELSE 'Normal'
    END as Fatigue_Risk_Category
FROM crew_incident i
JOIN [crew_scheduling ] s ON i.Crew_id = s.Crew_id
-- Ensures we only sum duty hours from the 7 days leading up to the incident
WHERE CAST(s.[Scheduled_date_for_starting] AS DATE) 
      BETWEEN DATEADD(day, -7, CAST(i.Incident_date AS DATE)) AND CAST(i.Incident_date AS DATE)
GROUP BY i.Incident_id, i.Crew_id, i.Incident_date, i.Severity_level;

--b) Are we successfully managing the frequency of "Critical" safety events month-over-month? (Aggregating Incident_Count by month and Severity_level).
--Shows the monthly progression of safety events.
CREATE VIEW vw_IncidentTrends AS
SELECT 
    -- FORMAT is perfect for SSMS to create a YYYY-MM string for Tableau
    FORMAT(CAST(Incident_date AS DATE), 'yyyy-MM') as Incident_Month,
    Incident_type,
    Severity_level,
    COUNT(Incident_id) as Incident_Count
FROM crew_incident
GROUP BY FORMAT(CAST(Incident_date AS DATE), 'yyyy-MM'), Incident_type, Severity_level;

--c) Which specific incident types are the primary drivers of our total network delay? (Ranking incident types by Total_Delay_Min and Avg_Delay).
---Quantifies the ripple effect of incidents on flight punctuality.
CREATE VIEW vw_SafetyDelayImpact AS
SELECT 
    Incident_type,
    COUNT(Incident_id) as Total_Incidents,
    SUM(Flight_delay_minutes) as Total_Delay_Min,
    ROUND(AVG(CAST(Flight_delay_minutes AS FLOAT)), 2) as Avg_Delay_Per_Incident
FROM crew_incident
GROUP BY Incident_type;

-- d) Which regional bases are the "Safety Leaders" vs. our highest risk hotspots? (Comparing normalized Incident_Rate_Per_Crew).
---Normalizes data to compare small bases and large hubs fairly.
CREATE VIEW vw_BaseSafetyHealth AS
WITH BaseIncidents AS (
    SELECT Crew_base_airport, COUNT(Incident_id) as Incident_Count
    FROM crew_incident
    GROUP BY Crew_base_airport
),
BaseCrew AS (
    SELECT Crew_base_airport, COUNT(DISTINCT Crew_id) as Unique_Crew_Count
    FROM [crew_scheduling ]
    GROUP BY Crew_base_airport
)
SELECT 
    i.Crew_base_airport,
    i.Incident_Count,
    c.Unique_Crew_Count,
    ROUND(CAST(i.Incident_Count AS FLOAT) / NULLIF(c.Unique_Crew_Count, 0), 4) as Incident_Rate_Per_Crew
FROM BaseIncidents i
JOIN BaseCrew c ON i.Crew_base_airport = c.Crew_base_airport;


--e) Which crew roles represent the most severe operational vulnerability during a crisis? (Role vs. Incident Type Matrix).
--Shows which crew roles are most affected by specific incident types.
CREATE VIEW vw_RoleSafetyMatrix AS
SELECT 
    s.Crew_role,
    i.Incident_type,
    COUNT(i.Incident_id) as Incident_Frequency,
    ROUND(AVG(CAST(i.Flight_delay_minutes AS FLOAT)), 2) as Avg_Delay_Impact
FROM crew_incident i
INNER JOIN [crew_scheduling ] s ON i.Crew_id = s.Crew_id
GROUP BY s.Crew_role, i.Incident_type;

```

-- # Area 4: Network Resiliency (Infrastructure Saturation)
```sql
--a)Which flight paths are our most unreliable?
--Shows the "reliability" of specific flight paths.
CREATE VIEW vw_RouteOTPAudit AS
SELECT 
    Origin_airport,
    Destination_airport,
    COUNT(Flight_id) as Total_Flights,
    SUM(CASE WHEN Delay_in_departure_min <= 15 THEN 1 ELSE 0 END) as On_Time_Flights,
    ROUND(CAST(SUM(CASE WHEN Delay_in_departure_min <= 15 THEN 1 ELSE 0 END) AS FLOAT) / 
          NULLIF(COUNT(Flight_id), 0) * 100, 2) as OTP_Percentage
FROM flight_ops
GROUP BY Origin_airport, Destination_airport
HAVING COUNT(Flight_id) > 10;


--b Are we pushing our human assets to the breaking point?
CREATE VIEW vw_CrewFatigueRisk AS
SELECT 
    Crew_id,
    Crew_duty_id,
    Actual_date_for_starting,
    Total_duty_hours,
    -- Calculates the sum of duty hours for the current record + the 6 preceding duty records for that crew member
    SUM(Total_duty_hours) OVER (
        PARTITION BY Crew_id 
        ORDER BY CAST(Actual_date_for_starting AS DATE) 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS Rolling7DayTotalHours,
    -- Categorizing risk based on typical aviation safety thresholds
    CASE 
        WHEN SUM(Total_duty_hours) OVER (PARTITION BY Crew_id ORDER BY CAST(Actual_date_for_starting AS DATE) ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) > 50 THEN 'High Risk'
        WHEN SUM(Total_duty_hours) OVER (PARTITION BY Crew_id ORDER BY CAST(Actual_date_for_starting AS DATE) ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) BETWEEN 40 AND 50 THEN 'Medium Risk'
        ELSE 'Safe'
    END AS FatigueStatus
FROM dbo.Crew_Scheduling;

--c) Which hub acts as the network's single point of failure? 
---Visualizes which airports are "bottlenecks" for the rest of the network.
CREATE VIEW vw_HubDependencyRisk AS
SELECT 
    Origin_airport,
    COUNT(DISTINCT Destination_airport) as Destinations_Served,
    ROUND(AVG(CAST(Delay_in_departure_min AS FLOAT)), 2) as Avg_Hub_Delay,
    ROUND(SUM(CAST(Delay_in_departure_min AS FLOAT)), 0) as Total_Network_Impact_Min
FROM flight_ops
GROUP BY Origin_airport;

--d) Where is our infrastructure failing to scale? (Month-over-Month Volume Growth vs. Delay Growth)
--We need the month from the scheduling table to calculate the growth metrics for the flights.
CREATE VIEW vw_SeasonalStrain AS
WITH HubMonthly AS (
    SELECT 
        f.Origin_airport,
        s.Roster_month,
        COUNT(f.Flight_id) as Flight_Count,
        SUM(CAST(f.Delay_in_departure_min AS FLOAT)) as Total_Delays
    FROM flight_ops f
    JOIN [crew_scheduling ] s ON f.Scheduled_departure_date = s.[Scheduled_date_for_starting]
    GROUP BY f.Origin_airport, s.Roster_month
),
GrowthStats AS (
    SELECT 
        Origin_airport,
        Roster_month,
        Flight_Count,
        LAG(Flight_Count) OVER (PARTITION BY Origin_airport ORDER BY Roster_month) as Prev_Flight_Count,
        Total_Delays,
        LAG(Total_Delays) OVER (PARTITION BY Origin_airport ORDER BY Roster_month) as Prev_Total_Delays
    FROM HubMonthly
)
SELECT 
    Origin_airport,
    Roster_month,
    Flight_Count,
    Total_Delays,
    ROUND(((CAST(Flight_Count AS FLOAT) - Prev_Flight_Count) / NULLIF(Prev_Flight_Count, 0)) * 100, 2) as Vol_Growth_Pct,
    ROUND(((CAST(Total_Delays AS FLOAT) - Prev_Total_Delays) / NULLIF(Prev_Total_Delays, 0)) * 100, 2) as Delay_Growth_Pct
FROM GrowthStats;
	

--e) What is the total "Incident Tax" currently burdening the entire network? (Aggregating incident delays to measure system friction).
CREATE VIEW vw_IncidentOpImpact AS
SELECT 
    ci.Incident_type,
    ci.Severity_level,
    COUNT(ci.Incident_id) AS IncidentCount,
    AVG(fo.Delay_in_departure_min) AS AvgDelayMinutes,
    SUM(fo.Delay_in_departure_min) AS TotalLostMinutes
FROM dbo.Crew_Incidents ci
JOIN dbo.Flight_Operations fo 
    ON ci.Crew_id = fo.Crew_id 
    AND ci.Incident_date = fo.scheduled_departure_date
GROUP BY ci.Incident_type, ci.Severity_level;

```

