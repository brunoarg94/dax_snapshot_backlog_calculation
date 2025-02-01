# Analyzing open risk backlog over time with Power BI and DAX

## Problem Statement

Recently, I faced an interesting challenge in tracking the evolution of open risks over time in a Cyber Security Risk Management process. The goal was to develop a dynamic and interactive visualization of the open risk backlog by leveraging periodic snapshots, allowing for historical trend analysis and better decision-making.

## Context

Our dataset consists of a list of risks with the following attributes:

- risk_id: Unique identifier for each risk.  
- risk_status: Either "Open" or "Closed".  
- identify_date: Date when the risk was identified.  
- remediate_date: Date when the risk was closed (if applicable).

This dataset contains simulated data and is a generic representation, focusing only on the essential fields required to solve the problem.

The key requirement was to create a backlog metric that dynamically calculates the number of open risks at any given quarter-end date. The metric needed to work in a Power BI line chart visual displaying the backlog trend over time.  

## Solution Using DAX

To achieve this, I created a DAX measure in Power BI that calculates the number of open risks at any specific quarter end. The formula is as follows:

-------------------------------------------------------------------------------------------------------------------------------------------------------------

```DAX
Risks Open Backlog = 
VAR CurrentDate = MAX(d_calendar[end_of_quarter])

RETURN
CALCULATE(
    DISTINCTCOUNT(f_risk_dataset[risk_id]),
    ALL(d_calendar),  -- Ignores any filters from the calendar table to ensure quarter-year analysis
    FILTER(
        f_risk_dataset,
        (f_risk_dataset[risk_status] = "Open" && f_risk_dataset[identify_date] <= CurrentDate) ||  -- Counts risks that are still open
        (f_risk_dataset[risk_status] = "Closed" && f_risk_dataset[remediate_date] > CurrentDate && f_risk_dataset[identify_date] <= CurrentDate) 
        -- Counts risks that were identified before and on the current quarter end date and closed after the current quarter end date
    )
)
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------

## Breakdown of the DAX Logic:

CurrentDate = MAX(d_calendar[end_of_quarter]): This variable captures the latest fiscal quarter-end date in the current context.  

ALL(d_calendar): Ensures that the measure is calculated independently of any applied filters on the calendar table.  

## Risk Filtering Logic:

- Includes risks identified on or before CurrentDate.  
- Counts risks that remain "Open" at CurrentDate.  
- Includes risks that were later "Closed" but were still open at CurrentDate (i.e., remediated after that date).  

## Snapshot Analysis Perspective

This Power BI measure represents a snapshot analysis because it captures the state of open risks at specific points in time (i.e., the end of each fiscal quarter).

- Point-in-Time Calculation: It does not track real-time changes but instead shows how many risks were open at each quarter-end.  
- Historical Trend Analysis: Even if risks are closed later, past snapshots remain unchanged, making this approach useful for understanding backlog evolution over time.  
- Fixed Reporting Perspective: The use of ALL(d_calendar) ensures that each quarter’s backlog is calculated independently, maintaining consistency for trend analysis.  

## Results and Insights

This measure allows the risk management team to analyze the backlog trend over time using a line chart in Power BI. The insights help in:

- Identifying periods where risk accumulation was high.
- Understanding whether remediation efforts are keeping up with newly identified risks.
- Making data-driven decisions to optimize risk resolution strategies.

## Conclusion

By leveraging Power BI and DAX, we successfully provided a clear and effective way to track open risk backlog over time.  
This has improved the visibility of risk trends, enabling the Cyber Security team to take proactive measures in addressing security threats efficiently.  
The approach demonstrates the power of DAX in handling time-based calculations, making Power BI a valuable tool for risk analysis and management.  
