# Aviation Operations Analysis: Aurora Pacific Airways Resilence Audit 
 A SQL driven diagonstic analysis of the existing flight operation data comprising for scheduling, crew bases, crew and incidents. Excel has been leveraged for data pre-processing, SQL for surfacing business questions and Tableau for executive-level storytelling, this project transforms 130k+records into a proactive risk management framework. 

## **Project Background**
Aurora Pacific Airways, is a low cost carrier in the United States, with its primary hub is San Franciso, serving Boston(BOS), Atlanta(ATL), Denver(DEN), SeattleSEA), Dallas Fort Worth(DFW), Los Angeles(LAX), Miami(MIA), New York City(JFK) and Chicago(ORD). In 2024, Aurora Pacific Airways (APA),  hit a critical operational plateau. Despite increasing flight volumes, the network began experiencing rising volatility and a noticeable degradation in On-Time Performance (OTP). The airline's leadership recognized that the existing schedule was no longer capable of absorbing minor shocks, but they lacked a unified view of where and how operational minutes were being lost.

To prevent system-wide failure, the Operations Department commissioned this diagnostic audit across four critical areas:
- **1)Network Operations:** To identify where the network is physically "hemorrhaging" minutes and pinpoint seasonal failure points.
- **2)Crew Logistics:** To evaluate crew base performance and identify role-based friction that prevents on-time departures.
- **3)Safety & Vulnerability:** To analyze the relationship between procedural safety and operational reliability.
- **4)Infrastructure Management:** To determine which airports of the APA network have reached a point of physical saturation.

# The Goal: Shift from "Reactive Firefighting" to "Proactive Resilience"—reclaiming lost minutes and bulletproofing the 2025 schedule.


- The Excel log sheet to record all the cleaning process can be found here [link]
- The SQL queries used to inspect the data for its quality for this analysis can be found here [link].
- Targed SQL queries regarding various business questions can be found here [link].

**4 Tableau dashboards for 4 groups of Stakeholders in the Operations Department of Aurora Pacific Airways can be found below:**
- For Network Planning & Scheduling Stakeholder -> to help them design a flight schedule that is time efficient and reliable, the dashboard can be found here <a href="https://public.tableau.com/app/profile/cynthia.napoleon/viz/Flight_Operations_AuroraPacificAirways/Dashboard1">[link]<a>
- For Crew Logistics & Resource Management -> to help them makes changes to ensure right people are at the right time without burning them out, the dashboard can be found here <a href="https://public.tableau.com/app/profile/cynthia.napoleon/viz/Flight_Operations_AuroraPacificAirways/Dashboard2">[link]<a>
- For Safety and Compliance -> to help them create plans that minimise operational risk and ensure procedural adherance, the dashboard can be found here <a href="https://public.tableau.com/app/profile/cynthia.napoleon/viz/Flight_Operations_AuroraPacificAirways/Dashboard3">[link]<a>
- For Ground Operations & Infrastructure ->to help them manage the physical "turn" of the aircraft and hub capacity, the dashboard can be found here <a href="https://public.tableau.com/app/profile/cynthia.napoleon/viz/Flight_Operations_AuroraPacificAirways/Dashboard4">[link]<a>

## **Data Structure & Initial Checks**
The companies main database structure as seen below consists of three tables: table1, table2, table3, table4, with a total row count of X records. A description of each table is as follows:

Table 2:
Table 3:
Table 4:
Table 5:
![Alt Text for Image](erd.png)

## **Executive Summary**
**Overview of Findings**
The 2024 Audit reveals an airline operating at a breaking point, with systemic fragility causing resiliency to plummet during seasonal peaks. While ORD emerged as the "Resiliency King" with an elite 78.49% recovery rate, the broader network is hemorrhaging time due to a staggering 40,710-minute "Operational Tax"—a figure equivalent to grounding three full aircraft for an entire month.

The audit identifies several critical failure points:
- **Infrastructure Saturation:** At core hubs like ATL, delays have spiked 35% faster than flight volume, proving the system has reached a physical ceiling.
- **The Second Officer Bottleneck:** Operational stability sank to its lowest point during junior flight-deck delays; a single late sign-in triggers a 79-minute system lag—the most lethal role-based delay in the fleet.
- **Human Capital Depletion:** A major risk factor was uncovered in crew health, with 47.7% of the workforce categorized as "High Risk" for fatigue, fueling the sick calls that account for 58% of all safety-related time loss.
- **The March Stress Spike:** Safety performance dipped to an annual low in March, which alone accounted for 22% of all yearly incidents.
The Domino Effect: Ground efficiency in laggard bases has hit a trough, "leaking" 5 minutes per turn and triggering mandatory overnight groundings at SFO due to the 18:00 "Breaking Point.”

## **Insights Deep Dive**

# Area 1: Network Operations (Network Congestion Audit)
- **A) Hub Bottlenecks: SFO Aircraft Rotation**
Analysis: SFO recorded 639 occurrences of Aircraft Rotation delays, representing 42% of all rotation-linked friction in the network. This metric confirms that SFO is a "downstream victim"; historically, the hub isn't failing locally, but is absorbing significant late arrivals, creating a "domino effect" that prevents the base from ever reaching a "clean" operational state.
- **B) Fleet Reliability: Tail N0076QX**
Analysis: In September 2024, Tail N0076QX recorded 10 cancellations, accounting for 15% of the entire fleet's monthly cancellations despite being only one aircraft. This "Problem Child" trend forces the NOCC to scramble for spare aircraft 51% more often than average, creating a performance drain on standby reserves.
- **C) Utilization: Port-to-Port Variance**
Analysis: SFO maintains a -74.37 min variance, while BOS and ORD "leak" roughly 5 minutes per turn (a 7% efficiency gap). This pattern reveals that East Coast ground process sluggishness is essentially "bleeding" potential flight time, resulting in a cumulative loss of capacity that could have supported two additional daily rotations.
- **D) Network Vulnerability: The 18:00 Breaking Point**
Analysis: Delays peak at 25.92 minutes at 18:00, a 38% increase over midday averages. This late-day spike is a critical failure point; historically, these minutes are unrecoverable due to curfew and crew rest rules, leading directly to the overnight grounding of 12% of the affected fleet.
- **E) Schedule Recovery: ORD Quick-Turn Efficiency**
Analysis: ORD achieved a 78.49% recovery rate, outperforming the network average by 22%. By successfully "Quick Turning" 8 out of 10 delayed aircraft, ORD serves as the gold standard for resiliency, proving that aggressive ground management can halt the propagation of network delays.
- **F) Route Predictability: SFO-LAX Volatility**
Analysis: The SFO-LAX corridor shows a Standard Deviation of 19.61, which is 45% higher than comparable short-haul routes. This "Wild Card" trend indicates that arrival times are highly inconsistent, creating a staffing nightmare where gate agents are underutilized during early arrivals and overwhelmed during late surges.

[Visualization specific to category 1]


## Area 2: Crew Logistics (Workforce Constraints)
- **A) Base Efficiency: DEN vs. UNK Variance**
Analysis: DEN shows a -0.05 hr duty variance, while "Unassigned" (UNK) crews show a 7-minute inefficiency gap (a 12% negative variance). This pattern proves that a lack of a structured home-base environment correlates with less disciplined duty transitions, adding nearly 400 hours of unproductive pay-time per month.
- **B) Role Friction: Captain Sign-in Bottlenecks**
Analysis: Captains face a 29.05-minute sign-in friction, contributing to 31% of all role-based departure delays. As the "Critical Path" role, this friction is the primary bottleneck; the flight is legally immobilized until the Captain clears, making this the highest-priority target for logistical streamlining.
- **C) Idle Time Costs: SFO/SEA Layover Overages**
Analysis: SFO and SEA average 200 extra hotel nights per month, representing 24% of the total network lodging budget. This "Schedule Mismatch" highlights a historical failure to align flight arrivals with minimum rest periods, resulting in millions in avoidable annual overhead.
- **D) Workload Equity: The ORD Inequity Gap**
Analysis: A 12.61-hour "Inequity Gap" exists between the top and bottom deciles of ORD Cabin Crew. This 25% variance in duty load is a leading indicator for burnout; historically, bases with this level of inequity see a 15% higher rate of labor grievances and unscheduled absences.
- **E) Shift Congestion: The Daytime "Congestion Tax"**
Analysis: The Daytime Shift (11,207 duties) carries a 28.92m delay, which is 5% higher than the "Morning Rush." This indicates that mid-day catering and security infrastructure is over-saturated, creating a hidden "tax" on every turn during the peak earning hours of the day.

[Visualization specific to category 2]


## Area 3: Safety & Vulnerability (Systemic Fragility)
- **A) Fatigue Correlation: The Procedural Failure Trend**
Analysis: 100% of the 211 incidents occurred under "Normal" fatigue ratings. This data debunks the "Exhaustion Myth," revealing that our safety risk is 100% procedural. The trend suggests that crews are making errors due to "complacency" or "process complexity" rather than physical tiredness.
- **B) Incident Trends: The March Stress Spike**
Analysis: March 2024 recorded 88 incidents, accounting for 22% of the annual total in a single month. This seasonal "Stress Spike" correlates with high volume, suggesting the safety culture becomes "fragile" when pushed to 90% capacity, requiring seasonal safety reinforcements.
- **C) Delay Impact: Sick Calls vs. Human Factors**
Analysis: Sick calls drain 3,558 minutes (58% of total safety delay volume), but Human Factors average a 60-minute delay per occurrence. This identifies a "Severity vs. Volume" pattern: sick calls bleed the system slowly, while human factors provide the "lethal" blows to the schedule.
- **D) Base Safety Health: The SFO Safety Gap**
Analysis: SFO’s incident rate (0.0737) is 18% higher than LAX. This normalized gap proves that SFO’s safety issues are not just a byproduct of size, but a localized "Safety Culture Gap" that is currently adding 5,000 extra minutes of friction per year.
- **E) Role Vulnerability: The Second Officer Bottleneck**
Analysis: A Second Officer late sign-in triggers a 79.38-minute system lag, which is 2.7x more severe than a Cabin Crew delay. This role-based vulnerability exposes a lack of redundancy in the junior flight deck, making the Second Officer the single most sensitive point of failure in the crew ecosystem.

[Visualization specific to category 3]


## Area 4: Network Resiliency (Infrastructure Saturation)
- **A) The Unreliable Corridor: BOS-DEN Failure**
Analysis: The BOS-DEN flight path operates at a 33.63% OTP, meaning it is failing 2 out of every 3 operations. This is the single lowest reliability score in the network, identifying this corridor as the primary driver of customer compensation claims and brand damage.
- **B) Human Capital Depletion: High-Risk Fatigue**
Analysis: 47.7% of the total workforce is categorized as "High Risk" for fatigue. This exhaustion trend is the "Invisible Tax" on the airline, directly fueling the high volume of sick calls that account for 58% of all safety-related delays.
- **C) The Systemic Bottleneck: MIA Impact**
Analysis: MIA accounts for 232,683 minutes of total network impact, representing ~15% of all network delays. MIA is the "lethal" bottleneck; a disruption here creates a ripple effect that is 3x larger than a disruption at any other secondary hub.
- **D) Infrastructure Saturation: The ATL Growth Disconnect**
Analysis: ATL's 16.79% volume increase triggered a 22.68% delay spike. This proves that delays are growing 35% faster than volume, a classic indicator of physical saturation; the hub can no longer "grow" without fundamentally breaking the schedule.
- **E) The Operational Tax: Cumulative Incident Friction**
Analysis: The network carries a 40,710-minute "Incident Tax." This total volume of lost time represents the equivalent of grounding 3 aircraft for an entire month, effectively shrinking the airline’s available assets and making the entire system increasingly brittle

[Visualization specific to category 4]


## Strategic Recommendations by Stakeholder Group:

- **1. Network Planning & Scheduling**
Corridor Rehabilitation (BOS-DEN): Immediate intervention is required for the BOS-DEN flight path. With a 33.63% OTP, this route requires a schedule "padding" adjustment or a shift in aircraft rotation to prevent it from remaining the network's primary reliability failure.
Buffer Realignment: Implement a mandatory "Late-Day Buffer" for flights arriving after the 18:00 Breaking Point. By adding 15 minutes of scheduled ground time during this window, the department can reduce the 12% overnight grounding rate currently caused by the late-day delay spike.
- **2. Crew Logistics & Resource Management**
Workload Equity Balancing: Redesign the bidding system at ORD to close the 12.61-hour Inequity Gap. By distributing duty hours more evenly, the department can lower the 47.7% High-Risk fatigue rate and reduce the resulting sick call volume that currently drains 3,558 minutes annually.
Role-Specific Redundancy: Introduce "Standby Second Officers" at high-friction hubs. Because a single Second Officer late sign-in scales into a 79-minute system lag (the highest in the network), providing junior flight-deck redundancy offers the highest ROI for delay mitigation.
- **3. Safety & Compliance**
Procedural Standardization: Transition safety focus from "Fatigue Management" to "Procedural Discipline." Since 100% of incidents occurred under normal fatigue ratings, the department should launch a targeted training campaign to address the "Safety Culture Gap" at SFO, where incident rates are 18% higher than at LAX.
Seasonal Risk Mitigation: Deploy "Safety Task Forces" during the March Stress Spike. With 22% of all annual incidents occurring in this single month, heightened oversight is required during seasonal volume surges to maintain system stability.
- **4. Ground Operations & Infrastructure**
Infrastructure Decoupling (ATL & MIA): Transition to a structural surge-management model at ATL, where delays are growing 35% faster than flight volume. Additionally, prioritize ground stability at MIA to protect the network from its 232,683-minute systemic ripple effect.
Efficiency Export (ORD/SFO Model): Formalize and export the "Quick Turn" protocols from ORD (78.49% recovery rate) to BOS and ORD. Standardizing these processes will stop the 5-minute-per-turn "time leakage" and recover lost capacity across the East Coast and Midwest corridors.


## Assumptions and Caveats
- **Data Lineage:** "UNK" (Unassigned) crew data is assumed to represent crews without a permanent base assignment rather than missing data.
- **Fatigue Thresholds:** Fatigue risk (47.7%) is based strictly on rolling 7-day duty hour totals. It does not account for individual rest quality.
- **Normalization:** All safety and delay rates were normalized by flight volume to allow for accurate comparisons between major hubs and smaller ports.
- **Incident Reporting:** This audit assumes a consistent reporting rate for "Critical" and "High" severity events across all regions.
