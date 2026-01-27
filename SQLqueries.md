```sql

```
-- # Area 1: Network Operations (Network Congestion Audit)

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





-- # Area 2: Crew Logistics (Workforce Constraints)
--
--
--
--
--
--


-- # Area 3: Safety & Vulnerability (Systemic Fragility)
--
--
--
--
--
--


-- # Area 4: Network Resiliency (Infrastructure Saturation)
--
--
--
--
--
--

