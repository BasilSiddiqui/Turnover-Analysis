# HR Turnover Analytics Dashboard (Power BI)

## Overview

This project demonstrates how to build a **Human Resources Turnover Analytics dashboard** in Power BI using a structured employee dataset and DAX measures to calculate key workforce metrics such as **Headcount, Joiners, Leavers, Net Movement, and Turnover Rate**.

The objective of this project is to help organizations monitor workforce changes over time, identify trends in employee departures, and support data-driven HR decision-making.

Employee turnover is a critical metric for organizations because high turnover can indicate underlying issues such as poor management practices, low employee engagement, compensation problems, or lack of career development opportunities. By implementing an analytics dashboard, companies can quickly identify these patterns and take proactive action.

---

# Dataset Structure

The dataset represents employee records and includes the following columns:

| Column                  | Description                                                          |
| ----------------------- | -------------------------------------------------------------------- |
| PERSON_NUMBER           | Unique employee identifier                                           |
| DEPT                    | Department the employee belongs to                                   |
| ASSIGNMENT_STATUS_TYPE  | Employee status (ACTIVE / INACTIVE)                                  |
| CONTRACT_TYPE           | Contract category (LIMITED / UNLIMITED)                              |
| DOB                     | Employee date of birth                                               |
| GENDER                  | Employee gender                                                      |
| ORIG_HIRE_DT            | Employee hire date                                                   |
| JOB_LEVEL               | Hierarchical level (Upper Management, Lower Management, Worker etc.) |
| ACTUAL_TERMINATION_DATE | Date the employee left the company                                   |
| ACTION_REASON           | Reason for leaving (Resignation, Retirement, End of Contract etc.)   |

Each row represents one employee.

---

# Data Model Design

A **Date Dimension Table** is created to support time-based analysis such as:

* Monthly headcount
* Quarterly turnover
* Yearly hiring trends

This follows the **Star Schema modelling principle**, which improves performance and ensures accurate time intelligence calculations.

### Date Table (DAX)

```DAX
Date =
VAR MinHire = COALESCE(MIN(Turnover[ORIG_HIRE_DT]), TODAY())
VAR MaxTerm = COALESCE(MAX(Turnover[ACTUAL_TERMINATION_DATE]), TODAY())

RETURN
ADDCOLUMNS(
    CALENDAR(
        DATE(YEAR(MinHire),1,1),
        DATE(YEAR(MaxTerm),12,31)
    ),
    "Year", YEAR([Date]),
    "Month No", MONTH([Date]),
    "Month", FORMAT([Date],"MMM"),
    "Month Year", FORMAT([Date],"MMM YYYY"),
    "Quarter", "Q" & FORMAT([Date],"Q")
)
```

### Important Step

After creating the date table:

**Mark it as the Date Table**

```
Model View → Select Date Table → Mark as Date Table → Choose Date Column
```

This ensures Power BI correctly performs time-based calculations.

---

# Relationships

Two relationships are created between the **Date table** and the employee dataset:

| From       | To                                | Status   |
| ---------- | --------------------------------- | -------- |
| Date[Date] | Turnover[ORIG_HIRE_DT]            | Inactive |
| Date[Date] | Turnover[ACTUAL_TERMINATION_DATE] | Active   |

The inactive relationship is activated in DAX when calculating **Joiners**.

---

# Key HR Metrics (DAX Measures)

## Start Date

Returns the first visible date in the filter context.

```DAX
Start Date =
MINX(VALUES('Date'[Date]), 'Date'[Date])
```

---

## End Date

Returns the last visible date in the filter context.

```DAX
End Date =
MAXX(VALUES('Date'[Date]), 'Date'[Date])
```

---

# Headcount Calculations

## Headcount at End of Period

Counts employees who were active at the end of the selected period.

```DAX
HC End =
VAR d = [End Date]
RETURN
CALCULATE(
    DISTINCTCOUNT(Turnover[PERSON_NUMBER]),
    Turnover[ORIG_HIRE_DT] <= d,
    OR(
        ISBLANK(Turnover[ACTUAL_TERMINATION_DATE]),
        Turnover[ACTUAL_TERMINATION_DATE] > d
    )
)
```

This includes employees who:

* Joined **before the end date**
* Have **not left yet**, or left **after the end date**

---

## Headcount at Start of Period

```DAX
HC Start =
VAR d = [Start Date]
RETURN
CALCULATE(
    DISTINCTCOUNT(Turnover[PERSON_NUMBER]),
    Turnover[ORIG_HIRE_DT] <= d,
    OR(
        ISBLANK(Turnover[ACTUAL_TERMINATION_DATE]),
        Turnover[ACTUAL_TERMINATION_DATE] > d
    )
)
```

---

## Active Headcount

```DAX
Active HC = [HC End]
```

Represents the **current workforce size** at the selected date.

---

# Workforce Movement Metrics

## Joiners

Counts employees hired within the selected period.

```DAX
Joiners =
CALCULATE(
    DISTINCTCOUNT(Turnover[PERSON_NUMBER]),
    USERELATIONSHIP(Turnover[ORIG_HIRE_DT], 'Date'[Date])
)
```

`USERELATIONSHIP` activates the **inactive hire-date relationship**.

---

## Leavers

Counts employees who left within the selected period.

```DAX
Leavers =
CALCULATE(
    DISTINCTCOUNT(Turnover[PERSON_NUMBER])
)
```

This uses the **termination date relationship**.

---

## Net Movement

Net workforce change during the period.

```DAX
Net Movement =
[Joiners] - [Leavers]
```

Interpretation:

| Value    | Meaning           |
| -------- | ----------------- |
| Positive | Workforce growth  |
| Negative | Workforce decline |

---

# Average Headcount

Average workforce size during the period.

```DAX
Average HC =
DIVIDE([HC Start] + [HC End], 2)
```

This method is widely used in HR analytics because it stabilizes turnover calculations.

---

# Turnover Metrics

## Turnover Rate

```DAX
Turnover % =
DIVIDE([Leavers], [Average HC])
```

Formula:

```
Turnover Rate = Employees Leaving / Average Headcount
```

This metric measures **employee attrition**.

---

# Voluntary Turnover

Voluntary turnover includes resignations initiated by employees.

```DAX
Voluntary Leavers =
CALCULATE(
    [Leavers],
    KEEPFILTERS(Turnover[Separation Type] = "Voluntary")
)
```

---

## Voluntary Turnover Rate

```DAX
Voluntary Turnover % =
DIVIDE([Voluntary Leavers], [Average HC])
```

This metric is extremely important because **voluntary exits often signal dissatisfaction**.

---

# Dashboard Insights

The dashboard enables companies to analyze:

### Workforce Stability

Track how employee count changes over time.

### Hiring Trends

Understand whether recruitment efforts are increasing workforce capacity.

### Attrition Patterns

Identify departments or job levels with higher turnover.

### Voluntary Resignations

Highlight employee engagement issues.

### Department-Level Analysis

Compare turnover across:

* IT
* HR
* Finance
* Operations
* Sales

---

# Business Value

Implementing this dashboard provides several benefits:

### Improved Workforce Planning

HR teams can anticipate hiring needs and prevent staff shortages.

### Reduced Recruitment Costs

By identifying causes of turnover, companies can address issues before employees leave.

### Management Accountability

Departments with high turnover can be investigated for leadership or culture problems.

### Strategic Decision Making

Executives gain visibility into workforce trends and organizational health.

---

# Example Visualizations

Typical visuals in the dashboard include:

* Headcount trend over time
* Monthly joiners vs leavers
* Turnover rate by department
* Voluntary vs involuntary turnover
* Workforce distribution by job level
* Gender distribution across departments

---

# Technologies Used

* **Power BI**
* **DAX**
* **Data Modeling**
* **HR Analytics**

---

# Possible Extensions

This project can be expanded with:

* Retention analysis
* Employee tenure analysis
* Predictive attrition modeling
* Machine learning turnover prediction
* Cohort analysis for new hires

---

# Conclusion

This project demonstrates how a well-designed data model combined with efficient DAX calculations can transform raw HR data into meaningful workforce insights.

The dashboard enables organizations to move from reactive HR management to **proactive workforce strategy**, ultimately improving employee retention and organizational performance.
