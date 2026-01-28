Dataset Field / Column Issue Identified Action Taken & Rationale
Flight Operations turnaround_time Negative values found. Midnight Rollover Correction: Identified negative values caused by departures occurring the day after arrival. Applied a 24-hour adjustment to normalize durations.
Flight Operations crew_id 4.1% missing values. Granularity Preservation: Generated surrogate analytical identifiers to maintain record granularity for flight-level analysis without losing 4% of the dataset.
Flight Operations crew_count 0 count for departed flights. Logic Validation: Cross-referenced flight status. If "Cancelled," 0 was retained. If "Departed," flagged as "Missing Crew Data" to avoid skewing average crew utilization metrics.
Crew Incident flight_delay_min Potential outliers (>180m). Outlier Profiling: Analyzed delays >3 hours. Since they represented <0.01% of data, they were retained as valid "Hard Outliers" to capture extreme operational disruptions.
Crew Incident incident_type "Unknown" codes. Standardization: Re-categorized "Unknown" entries as "Other" to maintain categorical consistency for visualization.
Crew Incident crew_id Blank / Null values. Integrity Check: Deleted records with null IDs where they served as Primary Keys to prevent join errors in the SQL database.
Crew Scheduling crew_role Blank values (0.18%). Imputation: Assigned a "Missing" label instead of "Other" to facilitate future source-system auditing while allowing current analysis to proceed.
Crew Scheduling total_duty_hours Negative values / Outliers. Format Correction: Investigated potential Time Zone conversion errors; flagged anomalous records for exclusion from aggregate sum calculations.
Crew Scheduling pairing_id Missing Data (20%). Reference Integrity: Retained as-is. As this is a system-generated ID, placeholders were avoided to prevent "Ghost Joins" during SQL analysis.
All Files All Columns Leading/Trailing spaces. Data Scrubbing: Used TRIM() functions across all text fields to ensure consistent matching during SQL JOIN operations.
All Files Date/Time Fields Inconsistent Formats. Temporal Standardization: Unified all date formats to YYYY-MM-DD and timestamps to 24-hour format for SQL compatibility.
