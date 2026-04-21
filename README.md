### Using CHATGPT for assistance with DAX

Here is a link to Housing Data https://drive.google.com/drive/folders/1Adn2w-4C8uUhTQU2f9ct_e8VcubzTP9i?usp=sharing

Open PowerBI and add these files. You will be using ChatGPT to help you write DAX to alter some tables to make a Power BI dashboard.

1. Open PowerBI and bring in the sheets for redfin_sales and interest_rates.

2. In the model view we can see there are no columns to join on. However we could make a month_year column in each table and join on that.
* Have ChatGPT help you use DAX to extract the month from the date column (Month of Period End) in the redfin_sales table and add it as a new column. Then, create a month_year column from the month column and the year column.  
* Next, create a month_year column in the interest_rates table. Make sure that it is formatted in the same way as the month_year column in the redfin_sales data.
* Use these newly created columns to create a relationship between the tables.

3. Each row of the interest_rates table has the interest rate for that date. Have ChatGPT help you create a new measure to get a single interest rate value for each month/year combination.

4. We want an updated affordability score for every month of data.  First, ask ChatGPT some suggestions for good formulas for this.  

Option1: Price-to-income adjusted by interest rate

Affordability Score =
DIVIDE(
    [Median Sale Price] * [Monthly Interest Rate],
    [Median Income]
)

Lower = better affordability

Option2: Index-bsed scoring

step 1: normalize components

Price Index =
DIVIDE([Median Sale Price], CALCULATE(AVERAGE([Median Sale Price]), ALL(Date)))

Rate Index =
DIVIDE([Monthly Interest Rate], CALCULATE(AVERAGE([Monthly Interest Rate]), ALL(Date)))

Income Index =
DIVIDE([Median Income], CALCULATE(AVERAGE([Median Income]), ALL(Date)))

step 2: combine

Affordability Score =
DIVIDE([Income Index], [Price Index] * [Rate Index])

great for trend charts

5. If ChatGPT didn't already suggest it, make a monthly payment calculator.  Create this column rounded to 2 decimal places.

   This DAX based on:

Loan amount = LoanAmount
Interest rate is annual (as decimal, e.g. 0.065 for 6.5%)
Loan term = 30 years (360 months)

Monthly Payment =
ROUND(
    VAR Loan = [LoanAmount]
    VAR MonthlyRate = [Average of MORTGAGE30US]
    VAR Months = 360
    VAR Payment =
        Loan *
        DIVIDE(
            MonthlyRate * POWER(1 + MonthlyRate, Months),
            POWER(1 + MonthlyRate, Months) - 1
        )
    RETURN Payment,
    2
)

6. Now, make a month by month inventory pressure score.  For this score, 0 should represent a stable inventory, a positive number should mean inventory is growing, and a negative number should mean inventory is shrinking. Ask ChatGPT to consider the inventory level, the Home Sold, and the New Listings is this score.

a month-by-month Inventory Pressure Score will need the following information:

1. Inventory level (snapshot supply)
2. New Listings (incoming supply)
3. Homes Sold (demand/absorption)

And the interpretation should be:

0 → stable market
> 0 → inventory building (supply increasing)
< 0 → inventory shrinking (demand > supply)

Pressure = Supply change - demand

Where: supply change = New listing
       demand = Homes sold

formula:
Inventory Pressure Score =
VAR NetFlow =
    [New Listings] - [Homes Sold]

VAR InventoryLevel =
    [Inventory]

RETURN
ROUND(
    DIVIDE(NetFlow, InventoryLevel),
    4
)

7. We want to use DAX to create a new Date table that contains columns for Quarter and Quarter Year as well as a Month Year column to make a relationship with our existing tables. Create this table and then make a relationship between it and the other two tables. You can check that you have set it up correctly by creating a visualization using the date table for the X-axis and the affordability index for the Y-axis. You may need to create a new column in your Date table to sort by.
