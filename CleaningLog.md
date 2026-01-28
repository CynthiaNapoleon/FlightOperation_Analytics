| Dataset | Field / Column | Issue Identified | Action Taken & Rationale |
| :--- | :--- | :--- | :--- |
| **Flight Operations** | `turnaround_time` | Negative values found. | **Midnight Rollover Correction:** Applied a 24-hour adjustment to normalize durations. |
| **Flight Operations** | `crew_id` | 4.1% missing values. | **Granularity Preservation:** Generated surrogate analytical identifiers to maintain record granularity. |
| **Flight Operations** | `crew_count` | 0 count for departed flights. | **Logic Validation:** Flagged anomalies as "Missing Crew Data" for analysis accuracy. |
| **Crew Incident** | `flight_delay_min` | Potential outliers (>180m). | **Outlier Profiling:** Retained as valid "Hard Outliers" (<0.01%) to capture extreme disruptions. |
| **Crew Incident** | `incident_type` | "Unknown" codes. | **Standardization:** Re-categorized "Unknown" entries as "Other". |
| **Crew Incident** | `crew_id` | Blank / Null values. | **Integrity Check:** Deleted records with null IDs to prevent join errors. |
| **Crew Scheduling** | `crew_role` | Blank values (0.18%). | **Imputation:** Assigned a "Missing" label to facilitate future auditing. |
| **Crew Scheduling** | `total_duty_hours` | Negative values / Outliers. | **Format Correction:** Flagged anomalous records for exclusion from aggregate sum calculations. |
| **Crew Scheduling** | `pairing_id` | Missing Data (20%). | **Reference Integrity:** Retained as-is; avoided placeholders to prevent "Ghost Joins". |
| **All Files** | All Columns | Leading/Trailing spaces. | **Data Scrubbing:** Used `TRIM()` functions across all text fields. |
| **All Files** | Date/Time Fields | Inconsistent Formats. | **Temporal Standardization:** Unified all date formats to `YYYY-MM-DD`. |
