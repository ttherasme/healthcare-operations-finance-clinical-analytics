# Power Query Transformation Documentation

This document logs the step-by-step ETL (Extract, Transform, Load) transformations applied via Excel Power Query to convert raw healthcare management data into optimized, clean reporting schemas.

## Global Pre-Processing (All Files)
1. **Source Connection:** Loaded CSV text files via Local Directory source pipelines using UTF-8 encoding.
2. **Promoted Headers:** Promoted first rows to column headers.
3. **Data Type Correction:** Explicitly defined string types for IDs, numbers for monetary fields (`cost`, `amount`), and standardized local dates (`YYYY-MM-DD`).

---

## 1. Operational Analytics Pipeline (`operational_analytics.xlsx`)
**Objective:** Consolidate healthcare accessibility and schedule management workflows.

1. **Base Table:** Started with `Appointments.csv`.
2. **Join Patients Data:** * Merged with `Patients.csv` matching on `patient_id` via a Left Outer Join.
   * Expanded: `gender`, `date_of_birth`, and `insurance_provider`.
3. **Derive Age Features:**
   * Added custom column `Age` derived from `date_of_birth` relative to runtime context.
   * Added Conditional Column `age_group`:
     * `if [Age] < 31 then "Young Adult"`
     * `if [Age] <= 64 then "Adult"`
     * `else "Senior"`
4. **Join Doctors Data:**
   * Merged with `Doctors.csv` matching on `doctor_id` via a Left Outer Join.
   * Expanded: `specialization`, `years_experience`, and `hospital_branch`.
5. **Output Selection:** Filtered and reordered specific tracking headers matching the operational layout.

---

## 2. Financial & Billing Pipeline (`financial_analytics.xlsx`)
**Objective:** Isolate transaction flows, collection pipelines, and revenue leaks.

1. **Base Table:** Started with `Billing.csv`.
2. **Join Treatments Data:**
   * Merged with `Treatments.csv` matching on `treatment_id` via a Left Outer Join.
   * Expanded: `appointment_id`, `treatment_type`, `cost`, and `treatment_date`.
3. **Join Appointments Data:**
   * Merged with `Appointments.csv` matching on `treatment.appointment_id` to `appointment_id` via a Left Outer Join.
   * Expanded: `patient_id`, `doctor_id`, and `status`.
4. **Cleanup Adjustments:** Verified formatting compatibility between currency fields (`amount` vs `cost`).

---

## 3. Clinical Analytics Pipeline (`clinical_analytics.xlsx`)
**Objective:** Track therapeutic throughput, diagnostic distributions, and provider analytics.

1. **Base Table:** Started with `Treatments.csv`.
2. **Join Appointments Data:**
   * Merged with `Appointments.csv` using `appointment_id` via a Left Outer Join.
   * Expanded: `patient_id`, `doctor_id`, `appointment_date`, and `status`.
3. **Join Patients Data:**
   * Merged with `Patients.csv` using `appointment.patient_id` to `patient_id`.
   * Expanded: `gender` and `date_of_birth`.
4. **Derive Clinical Demographics:**
   * Calculated clinical age metrics and added conditional grouping matching the operational schema `Age_Group` (*Young Adult / Adult / Senior*).
5. **Join Doctors Data:**
   * Merged with `Doctors.csv` using `appointment.doctor_id` to `doctor_id`.
   * Expanded: `specialization` and `hospital_branch`.