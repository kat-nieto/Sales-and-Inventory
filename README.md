## Problem Statement:

The task at hand is to analyze a detailed inventory dataset to determine which products should remain in the company's inventory and which should be removed. The dataset contains information on product characteristics, sales data, customer behavior, and inventory details across multiple regions. The goal is to make data-driven decisions on inventory management to optimize product assortment, improve profitability, and reduce unnecessary stock holdings.

This involves assessing factors such as profit margins, inventory levels, customer demand, and geographical sales patterns to identify products that are underperforming or overstocked, and those that are most likely to continue contributing to revenue.
The dataset includes various attributes such as product details (e.g., name, description, cost, price), sales information (e.g., order quantities, customer data, pricing), and inventory data (e.g., stock levels, warehouse details). By examining these variables, the objective is to assess product performance across different regions, customer segments, and sales patterns.

## Dashboard Link:

https://bit.ly/Sales_and_Inventory

## The Problem:
The company is facing challenges in managing its product portfolio and inventory effectively. With a large number of products across various regions and customer segments, the business needs to identify which products are worth keeping in inventory and which should be discontinued. There is a risk of overstocking slow-moving products or understocking popular products, which can lead to lost sales and increased costs.

## Key Questions:

Which products are most profitable and should be retained in the inventory?

Which products are underperforming in terms of sales, profitability, or demand, and should be discontinued or removed?

What is the optimal stock level for each product based on historical sales, profit margins, and customer demand?

How do geographical factors (e.g., region, country) influence product performance and sales patterns?

What is the relationship between the Product list price, Product standard cost, Order item quantity, Per unit price, Total item quantity for each product? 

What are the key metrics (e.g., stock value, sell-through rate, stock holding cost) that can help in making informed decisions about product assortment?

How can the company improve its inventory turnover, reduce stock shrinkage, and optimize warehouse operations?

## Objectives:

1. Profitability Analysis: Evaluating products based on profit margins, total profit, and revenue generation.

2. Inventory Optimization: Determining the optimal stock levels (e.g., order item quantity, total item quantity) and identifying excess inventory or understocked products.

3. Sales and Demand Patterns: Analyzing sales trends, customer demand, and historical order data to predict which products are most likely to continue generating revenue and which may need to be discontinued.

4. Geographical and Demographic Insights: Understanding how factors such as region, country, and customer profile impact product performance and inventory decisions.

5. Cost Analysis: Assess inventory-related costs, including stock holding costs, stock shrinkage, and the cost of unsold inventory, to make cost-effective inventory management decisions.

6. Portfolio Management: Establish clear criteria for product assortment to ensure the product portfolio aligns with both profitability goals and customer demand, ensuring the right mix of products is available in each region.

7. Improvement of Operational Efficiency: Recommend strategies for reducing inventory waste, improving stock turnover rates, and enhancing the overall efficiency of inventory management processes.

## Steps followed

• Step 1: Load data into Power BI, dataset is a csv file.

• Step 2: Open power query and in the factOrder table added five conditional columns to create the indexes for OrderCode, CustomerCode, EmplyeeCode, ProductCode, WareHouseCode.

• Step 3: Duplicate four times the fact table factOrder and create four dimension tables: dimCustomer, dimEmployee, dimProduct, dimWareHouse.

• Step 4: Remove the extra columns from the fact table, in the dimension tables remove duplicates, and change data type for the corresponding columns.

• Step 5: Create a new query and use the advance editor and type the code to create a dimCalendar table.

        let
    MinDate = List.Min (factOrder [OrderDate]),
    MaxDate = List.Max (factOrder [OrderDate]),
    YearMinDate = Date.Year (MinDate),
    YearMaxDate = Date.Year (MaxDate),
    TotalDays = Duration.Days (Date.FromText ("12/31/" & Text.From (YearMaxDate)) - Date.FromText ("01/01/" & Text.From(YearMinDate))) + 1 ,
    ListDays = List.Dates (Date.FromText ("01/01/" & Text.From (YearMinDate)) , TotalDays, #duration (1,0,0,0)),
    CalendarTable = Table.FromList (ListDays, Splitter.SplitByNothing (), null, null, ExtraValues.Error),
    #"Inserted Year" = Table.AddColumn(CalendarTable, "Year", each Date.Year([Column1]), Int64.Type),
    #"Inserted Month" = Table.AddColumn(#"Inserted Year", "Month", each Date.Month([Column1]), Int64.Type),
    #"Inserted Month Name" = Table.AddColumn(#"Inserted Month", "Month Name", each Date.MonthName([Column1]), type text),
    #"Inserted First Characters" = Table.AddColumn(#"Inserted Month Name", "First Characters", each Text.Start([Month Name], 3), type text),
    #"Inserted Merged Column" = Table.AddColumn(#"Inserted First Characters", "Month/Year", each Text.Combine({[First Characters], Text.From([Year], "en-US")}, "/"), type text),
    #"Changed Type" = Table.TransformColumnTypes(#"Inserted Merged Column",{{"Column1", type date}}),
    #"Renamed Columns" = Table.RenameColumns(#"Changed Type",{{"Column1", "Date"}}),
    #"Inserted Week of Year" = Table.AddColumn(#"Renamed Columns", "Week of Year", each Date.WeekOfYear([Date],  Day.Monday))
        in
    #"Inserted Week of Year"

• Step 6: In Power BI check the model view, choosing a start schema and double check the table relationship, cardinality, and cross filter direction.

• Step 7: Create the Sales calculated measures with DAX:

        Volume Order = COUNTROWS(factOrder)

        Units Sold = SUM(factOrder[OrderItemQuantity])

        Units Received = SUM(factOrder[TotalItemQuantity])

        Total Standard Price = SUM(dimProduc [ProductListPrice])

        Total Deal Price = SUM(factOrder[PerUnitPrice])

        Revenue = [Units Sold] * [Total Standard Price]

        Total Cost = SUM(dimProduct[ProductStandardCost])* [Units Sold]

        Profit = [Revenue] - [Total Cost]

        Profit Margin = DIVIDE([Profit], [Revenue])

        Sell-Through Rate = DIVIDE([Units Sold], [Units Received], 0) 

        Customer Credit Limit = SUM(dimCustomer[CustomerCreditLimit])

• Step 8: Create the Inventory calculated measures with DAX:

        Inventory Value = [Units Received] * [Total Deal Price]

        Inventory holding Cost = DIVIDE([Total Cost], [Inventory to Sell])

        Inventory to Sales Ratio = DIVIDE([Inventory to Sell], [Units Sold]) 

        Inventory Shrinkage = DIVIDE([Inventory holding Cost], [Inventory Value] ,0) 

        Portafolio = DISTINCTCOUNT(dimProduct[ProductName])

        Dead Stock = DIVIDE ([Inventory to Sell], [Units Received], 0)

• Step 9: Create a new page for the Tooltip, and design the Gague chart plus the Gaguer axis, the following DAX expression was written:

        Tooltip- Sell-Through Rate = IF( [Sell-Through Rate] > 1 , [Sell-Through Rate], 1)

        Color Sell-Through Rate = IF([Sell-Through Rate] > 1, "#0FF00B" , IF( [Sell-Through Rate] = 1, "#84898D", "#F61104" ))

![Tooltip](https://github.com/user-attachments/assets/104017a5-4bce-49f7-95de-6fd42761469c)


• Step 10: In the report view, develop the visualization using a canvas background previously designed in Power Point

• Step 11: Visual filter (Slicer) was added for the year in the Sales page.

• Step 12: Five cards new were added to the Sales page representing Revenue, Cost, Profit, Profit Margin, Sell-Through Rate.

• Step 13: Build different type of charts: line and clustered column chart, decomposition tree, stacked bar chart, and KPI.

• Step 14: Insert a botton to redirect to the Inventory page.

• Step 15: Visual filter (Slicer) was added for the year in the Inventory page.

• Step 16: Five cards new were added to the Inventory page representing Stock Value, Stock Holding Cost, Stock to Sales Ratio, Stock Shrinkage, Portafolio.

• Step 17: Bookmarks and icon buttons were created to view the different type of charts built: line and stacked column chart, stacked bar chart, waterfall chart, donut chart, map, and matrix.

## Snapshop of Dashboard (Power BI Service)

![Page 1](https://github.com/user-attachments/assets/8b918b42-ad9d-49e2-924f-7f7b0cc906a4)

![Page 2](https://github.com/user-attachments/assets/07b43aa9-cae5-42ff-9c4f-635a3bf5bfb9)
