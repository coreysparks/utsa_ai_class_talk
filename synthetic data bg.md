# Context Transfer: Corey Sparks, Synthetic Data Experience

## Purpose

This document provides background from previous ChatGPT conversations to support development and revision of a presentation about synthetic data. Treat this as project context, distinguishing completed work from conceptual or proposed projects.

## Professional Background

Corey Sparks, PhD, is an applied demographer and data scientist with extensive experience in administrative data analysis, statistical modeling, data engineering, and analytical workforce development. His background spans academic research, public sector data systems, and applied data science training.

## 1. Synthetic Corrections Administrative Data

Designed and generated synthetic individual-level corrections administrative data for use in analytical training.

Project dataset: `correctional_trends_synthetic_v4.csv`

Features included:

- Approximately 20 correctional facilities with different population characteristics and security classifications.
- Individual demographic characteristics, including gender and race.
- Admission and parole events with associated dates.
- Facility constraints governing admissible population characteristics.
- Temporal patterns, including seasonality in admissions.
- Derived variables supporting descriptive, longitudinal, and facility-level analysis.
- Additional sampled data with intentionally introduced missing values.

The objective was to produce a plausible, internally coherent administrative dataset suitable for demonstrating realistic analytical workflows without using confidential individual records.

### Educational Applications

Used the synthetic corrections data across three instructional courses:

1. Introduction to Data Visualization Using R for Corrections Analysts.
2. Introduction to Dashboards Using R for Corrections Analysts.
3. Introduction to Quarto and Reproducible Reporting for Corrections Analysts.

Applications included exploratory data analysis, visualization, interactive dashboards, KPI development, parameterized reporting, and reporting automation.

Tools included R, tidyverse, ggplot2, Quarto, Shiny, Plotly, and gt.

## 2. Synthetic Healthcare Administrative Data

Designed a synthetic Medicaid administrative data architecture with approximately 10,000 beneficiaries.

The proposed structure incorporated eligibility histories, healthcare claims, inpatient claim headers, inpatient service lines, and an analytical inpatient stay fact table.

Key design considerations included relational integrity, valid linkages between beneficiaries and claims, and realistic healthcare utilization events.

Project implementation status should be verified before describing this work as a completed synthetic data system.

## 3. Statistical Simulation and Measurement Error

Conducted research on the implications of measurement error in American Community Survey estimates for regression coefficient estimation.

Related methodological work includes:

- Bayesian measurement error modeling.
- Regression analysis with error-prone covariates.
- Modeling uncertainty in demographic estimates.
- Simulation-based evaluation of statistical methods.
- Demographic projection models.

This work provides methodological experience relevant to synthetic data generation, but should be distinguished from direct production of synthetic administrative microdata.

## 4. Analytical and Data Engineering Applications

Synthetic data has been used to demonstrate complete analytical pipelines:

- Data preparation and feature engineering.
- Relational data transformations.
- Data quality validation.
- Statistical summaries.
- Dashboard development.
- Parameterized and automated reporting.

## 5. Methodological Strengths

The most directly documented capabilities include constraint-based synthetic data generation, simulated temporal processes, structured administrative datasets, intentional missingness, and educational applications of synthetic data.

Do not assume experience with GANs, VAEs, differential privacy, or quantitative disclosure risk validation without additional supporting evidence.

## Guidance for Revising the Presentation

Use this background to connect Corey's practical experience with broader synthetic data concepts.

Emphasize the distinction between simulated data generated from analyst-specified rules and statistically calibrated synthetic data generated from real observations.

Where relevant, discuss analytical utility, structural validity, statistical fidelity, and privacy risk as separate dimensions of evaluation.

Do not attribute methods or completed implementations to Corey unless supported by his actual project history or presentation materials.