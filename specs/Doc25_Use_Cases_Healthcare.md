## Document 25 ‒ Use Cases – Healthcare

### Purpose

To describe how AionFM can support forecasting in healthcare operations, including patient arrivals, bed occupancy, staffing demand, procedure volume and supply usage.  Healthcare environments are characterised by high uncertainty, seasonality (flu season, holiday dips), regulatory constraints and capacity limits.  This document provides guidance on data sources, covariates and scenario applications.

### Intended Audience

Healthcare planners, hospital administrators, operations researchers and data scientists.

### Key Sections

* **Healthcare operations dynamics** – arrivals, lengths of stay, resource usage.
* **Data sources** – patient records, electronic health records (EHR), scheduling systems.
* **Covariates** – seasonality, disease incidence, staffing schedules, policy changes.
* **Capacity constraints** – bed and staff availability, regulatory requirements.
* **Scenario planning** – surge scenarios and planning tools.

### Healthcare Operations Dynamics

Hospitals must forecast:

* **Patient arrivals** – emergency department visits, elective admissions, transfers.  Arrival rates vary with season, day of week and public events.
* **Lengths of stay** – distribution of patient stays influences bed occupancy.  Longer stays may indicate severity or delays in discharge.
* **Procedure volume** – scheduled surgeries, imaging, outpatient visits.
* **Resource usage** – bed occupancy, staffing hours, consumable supplies.

### Data Sources

Use data from:

* **Electronic health records (EHR)** – timestamps for admissions, discharges, procedures.
* **Scheduling systems** – planned procedures, staffing rosters.
* **Public health data** – flu incidence, COVID‑19 case counts, disease outbreaks.
* **Policy databases** – changes in regulations, reimbursement policies, hospital capacity guidelines.

### Covariates

Relevant covariates include:

* **Seasonality indicators** – week of year, holidays, school breaks.
* **Disease incidence** – number of flu cases, COVID‑19 case rates, other epidemics.
* **Staffing schedules** – planned staffing levels and shift patterns.
* **Policy changes** – new regulations affecting elective surgery volume or bed allocation.
* **Weather** – extreme temperatures may drive emergency visits.

### Capacity Constraints

Hospitals operate under strict capacity limits: number of beds, ICU beds, ventilators, staff.  AionFM can incorporate capacity constraints into its forecasts via constraint‑aware decoding and scenario filtering.  For example, bed occupancy predictions cannot exceed physical bed counts.  Forecasts should include quantiles and scenarios to plan staffing and bed management under surges.

### Scenario Planning

Healthcare planners can use scenarios to:

* **Plan for surges** – simulate high‑arrival scenarios during flu season or disease outbreaks; identify when capacity will be overwhelmed and plan surge staffing or elective surgery cancellations.
* **Evaluate policy impacts** – model the effect of changing admission policies or reimbursement rules on procedure volume and occupancy.
* **Supply planning** – simulate increased supply usage during disease waves; ensure adequate inventory of masks, PPE and medications.
* **Staffing** – plan staffing buffers based on high‑demand scenarios; identify periods with high risk of understaffing.

AionFM’s probabilistic forecasting and scenario generation help healthcare systems build resilient, data‑driven operational plans.