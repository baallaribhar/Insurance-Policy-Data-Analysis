# Insurance-Policy

# Insurance Data Analysis using SQL

##  Project Overview

This project focuses on analyzing insurance data using **MySQL** to answer key business questions related to customers, policies, premiums, policy status, demographics, and policy trends.

The project involved creating relational database tables, importing and cleaning insurance datasets, establishing relationships between tables using `Customer_ID`, and writing SQL queries to generate business insights.

The analysis demonstrates practical SQL skills including **database creation, data importing, data cleaning, joins, aggregations, conditional logic, subqueries, CTEs, window functions, and trend analysis**.



##  Project Objectives

- Create a relational insurance database using MySQL.
- Import customer, policy, claims, payment, and additional insurance data.
- Clean and prepare the imported data for analysis.
- Analyze customer and policy information.
- Identify active, lapsed, and terminated policies.
- Analyze policy participation across age groups and genders.
- Compare different policy types.
- Analyze premium trends and growth rates.
- Identify customers with multiple policies.
- Identify customers who do not have any policy.
- Generate insights to support insurance business analysis.



##  Database Tables

The project uses the following tables:

### 1. Customer_Information

Contains customer-level information such as:

- Customer ID
- Gender
- Age / Age Group
- Location
- Occupation
- Other customer attributes

### 2. Policy_Details

Contains information related to insurance policies such as:

- Policy ID
- Customer ID
- Policy Type
- Policy Status
- Coverage Amount
- Premium Amount
- Start Year
- End Year

### 3. Claims

Contains information related to insurance claims such as:

- Customer ID
- Policy ID
- Claim Amount
- Claim Date
- Settlement Date
- Claim-related information

### 4. Payment_History

Contains information related to policy payments and payment history.
###  5. Additional_Fields

Contains additional insurance-related attributes used for analysis.



### Table Relationships

The primary relationship between the customer and policy tables is established using:

```sql
Customer_Information.Customer_ID
        ↓
Policy_Details.Customer_ID



### SQL queries

create table claims (
Claim_ID varchar (20) primary key,
Date_of_Claim date,
Claim_Amount decimal (15,2),
Claim_Status varchar (30),
Reason_for_Claim text,
Settlement_Date date,
Policy_ID varchar (20),

foreign key (Policy_ID)
references policy_details (Policy_ID)
);

create table payment_history(
Payment_ID	varchar(20) primary key,
Date_of_Payment date,	
Amount_Paid	decimal (15,2),
Payment_Method varchar(50),
Payment_Status varchar(30),
Policy_ID varchar(20),

foreign key (Policy_ID)
references policy_details (Policy_ID)
);

create table additional_fields(
Agent_ID varchar(20),
Renewal_Status varchar(30),	
Policy_Discounts decimal(10,2),
Risk_Score int,
Policy_ID varchar(20) primary key,

foreign key (Policy_ID)
references policy_details (Policy_ID)
);

describe claims;

ALTER TABLE Claims
MODIFY COLUMN Settlement_Date VARCHAR(20) NULL;

SELECT COUNT(*) AS Total_Records
FROM claims;

SET SQL_SAFE_UPDATES = 0;

UPDATE Claims
SET Settlement_Date = NULL
WHERE TRIM(Settlement_Date) = '';

ALTER TABLE Claims
MODIFY COLUMN Settlement_Date DATE NULL;

#total number of customers in the policy dataset
SELECT COUNT(*) AS Total_Customers
FROM customer_information;

#total number of policies issued
SELECT COUNT(*) AS Total_Policies
FROM policy_details;

#cutomers having more than 1 policy
SELECT 
Customer_ID,
COUNT(Policy_ID) AS Number_of_Policies
FROM policy_details
GROUP BY  customer_ID
HAVING COUNT(Policy_ID)> 1;

#how many distinct customers have policies
SELECT COUNT(DISTINCT Customer_ID)
FROM policy_details;

#how many customers don't have policies
SELECT COUNT(*) AS Customers_without_Policy
FROM Customer_Information c
LEFT JOIN Policy_Details p
    ON c.Customer_ID = p.Customer_ID
WHERE p.Customer_ID IS NULL;

SELECT COUNT(*) AS Cutomers_without_Policy
FROM Policy_Details p
RIGHT JOIN Customer_Information c
ON p.Customer_ID = c.Customer_ID
WHERE p.Customer_ID IS NULL;


#Total claim amount generated from all polices
SELECT SUM(Claim_Amount) AS Total_Claim_Amount
FROM claims;

#Average coverage amount per policy
SELECT AVG(Coverage_Amount) AS Average_Coverage_Amount
FROM policy_details;

#Average premium amount collected per policy
SELECT AVG(Premium_Amount) AS Average_Premium_Amount
FROM policy_details;

#percentage of policies currently active
SELECT 
    ROUND(
        100.0 * SUM(CASE WHEN Status = 'Active' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS Active_Policy_Percentage
FROM Policy_Details;


#number of active, lapsed, terminated policeis
SELECT Status,
COUNT(*) AS Policy_Count
FROM policy_details
GROUP BY Status;


#which policy status has the highest number
SELECT status,
COUNT(*) AS Policy_Count
FROM policy_details
GROUP BY Status
ORDER BY Policy_Count DESC
LIMIT 1;

select * from policy_details;

#ratio between active and inactive
SELECT
SUM(CASE when Status ='Active' then 1 else 0 end) AS Active_Polices,
SUM(CASE WHEN Status <> 'Active' THEN 1 ELSE 0 END) AS Inactive_Polices,

ROUND(
SUM(CASE when Status ='Active' then 1 else 0 end) /
NULLIF(SUM(CASE WHEN Status <> 'Active' THEN 1 ELSE 0 END),0),2)
AS Active_to_Inactive_Ratio
FROM policy_details;

SELECT
    Active_Policies,
    Inactive_Policies,
    ROUND(
        Active_Policies / NULLIF(Inactive_Policies, 0),
        2
    ) AS Active_to_Inactive_Ratio
FROM (
    SELECT
        SUM(CASE WHEN Status = 'Active' THEN 1 ELSE 0 END) AS Active_Policies,
        SUM(CASE WHEN Status <> 'Active' THEN 1 ELSE 0 END) AS Inactive_Policies
    FROM Policy_Details
) AS Policy_Status;

#age group with highest number of policies
SELECT
c.Age_Group,
COUNT(p.Policy_ID) AS Policy_Count
FROM customer_information c
JOIN policy_details p
ON c.Customer_ID = p.Customer_ID
GROUP BY c.Age_Group
ORDER BY Policy_Count DESC
LIMIT 1;


#top 3 age groups by policy count
SELECT 
c.Age_Group,
COUNT(p.Policy_ID) AS Policy_Count
FROM 
customer_information c
JOIN policy_details p
ON
c.Customer_ID = p.Customer_ID
GROUP BY Age_Group
ORDER BY Policy_Count DESC
LIMIT 3;


#gender with the highest policy participation
SELECT c.Gender,
COUNT(p.Policy_ID) AS Policy_Count
FROM customer_information c
JOIN policy_details p 
ON c.Customer_ID =p.Customer_ID
GROUP BY Gender
ORDER BY Policy_Count DESC
LIMIT 1;


#difference between male and female policy count
SELECT
SUM(CASE WHEN c.Gender='Male' THEN 1 ELSE 0 END) AS Male_Policies,
SUM(CASE WHEN c.Gender='Female' THEN 1 ELSE 0 END) AS Female_Polices,

ABS(
SUM(CASE WHEN c.Gender='Male' THEN 1 ELSE 0 END) -
SUM(CASE WHEN c.Gender='Female' THEN 1 ELSE 0 END) 
) AS Difference

FROM customer_information c
JOIN policy_details p
ON c.Customer_ID = p.Customer_ID;

#another way

SELECT
  ABS(Male_Policies - Female_Policies) AS Difference
FROM(
SELECT
    SUM(CASE WHEN c.Gender = 'Male' THEN 1 ELSE 0 END) AS Male_Policies,
    SUM(CASE WHEN c.Gender = 'Female' THEN 1 ELSE 0 END) AS Female_Policies
FROM Customer_Information c
JOIN Policy_Details p
    ON c.Customer_ID = p.Customer_ID) AS Gender_Count;
    
    
    #policy type with maximum number
    select Policy_Type,
    Count(*) as Policy_Count
    from policy_details
    group by Policy_type
    order by Policy_Count desc
    limit 1;
    
    
    #policy type with minimum number
    select Policy_Type,
    count(*) as Policy_Count
    from policy_details
    group by Policy_Type
    order by Policy_Count asc
    limit 1;
    
    
    #compare auto and health policy counts
    select Policy_Type,
    count(*) as Policy_Count
    from policy_details
    where Policy_Type in('Auto', 'Health')
    group by Policy_Type;
    
    
    #total number of policies across all policy types
    select
    count(*) as Total_Polices
    from policy_details;
    
    
    #average premium growth rate over all years
    WITH Yearly_Premium AS (
    SELECT
        Start_Year,
        AVG(Premium_Amount) AS Avg_Premium
    FROM Policy_Details
    GROUP BY Start_Year
),
Premium_Growth AS (
    SELECT
        Start_Year,
        Avg_Premium,
        LAG(Avg_Premium) OVER (ORDER BY Start_Year) AS Previous_Avg_Premium
    FROM Yearly_Premium
)
SELECT
    ROUND(
        AVG(
            (Avg_Premium - Previous_Avg_Premium)
            / Previous_Avg_Premium * 100
        ),
        2
    ) AS Average_Premium_Growth_Rate
FROM Premium_Growth
WHERE Previous_Avg_Premium IS NOT NULL;


#is premium growth increasing or decreasing
WITH Yearly_Premium AS (
    SELECT
        Start_Year,
        AVG(Premium_Amount) AS Avg_Premium
    FROM Policy_Details
    GROUP BY Start_Year
)
SELECT
    Start_Year,
    ROUND(Avg_Premium, 2) AS Avg_Premium,
    ROUND(
        (Avg_Premium - LAG(Avg_Premium) OVER (ORDER BY Start_Year))
        / LAG(Avg_Premium) OVER (ORDER BY Start_Year) * 100,
        2
    ) AS Premium_Growth_Rate
FROM Yearly_Premium
ORDER BY Start_Year;


#difference between highest and lowest growth rates
WITH Yearly_Premium AS (
    SELECT
        Start_Year,
        AVG(Premium_Amount) AS Avg_Premium
    FROM Policy_Details
    GROUP BY Start_Year
),
Premium_Growth AS (
    SELECT
        Start_Year,
        (
            Avg_Premium -
            LAG(Avg_Premium) OVER (ORDER BY Start_Year)
        )
        /
        LAG(Avg_Premium) OVER (ORDER BY Start_Year) * 100
        AS Growth_Rate
    FROM Yearly_Premium
)
SELECT
    ROUND(MAX(Growth_Rate) - MIN(Growth_Rate), 2)
    AS Difference_Highest_Lowest_Growth
FROM Premium_Growth
WHERE Growth_Rate IS NOT NULL;


#yearly trends of policies ending from 2016-2034
SELECT
    End_Year,
    COUNT(*) AS Policies_Ending
FROM Policy_Details
WHERE End_Year BETWEEN 2016 AND 2034
GROUP BY End_Year
ORDER BY End_Year;
