Data Analysis

The given data is CRM data with all the Users, Sales, Orders, Products, LineOfBusiness, Accounts, Orderlines, ProductSourceTokens, Resources, and Skus data.
This data improves User relationships, enhances sales Processes.
Accounts: Contains information about individual customer accounts.
Users: Linked to accounts, storing user-specific data within an account.
LineOfBusiness: Defines different business segments or units.
Orders: Records of purchases made by customers, linked to accounts and products.
Orderlines: Details of items within an order, linked to orders and products.
Products: Information about products available for sale.
ProductSourceTokens: May be used for product tracking or source identification.
Resources: Could include digital or physical resources associated with products or services.
Skus: Stock Keeping Units, unique identifiers for products.
There is One-to-Many relationship between Accounts and orders, Many-to-One relationship between Orders and Accounts, One-to-Many relationship between Orderlines and Orders, Many-to-One relationship between Orderlines to Products,One-to-Many relationship between Products to Skus.
There are 18 columns in Accounts table with 10 rows in it. and We can use AccountId to partition data.
There are 4 columns with 16 rows in the LineOfBusiness table. and can be partitioned by using LineOfBusinessId.
There are 17 columns with 1135 rows in Orderlines and can be partitioned by using OrderLineId.
There are 25 columns with 739 rows in Orders table and can be partitioned by using OrderTXNID.
There are 28 columns with 132 rows in Products table and can be partitioned by using ProductId by removing some of the rows without the ProductId.
There are 4 columns with 107 rows in ProductSourceTokens table and can be partitioned by using ProductId or ProductTokenId.
There are 19 columns with 105 rows in Resources table.
There are 26 columns with 11 rows in Users table and can be partitioned by using UserId or Email.
There are 17 columns with 145 rows in Skus table and can be partitioned by using Sku.
Note:
Some of the provided CSV files have an issue with the data. If the data in the column has a comma in t it is considered it has two separate columns because of the comma separator. It would have been better if the column data was mentioned in the double quotes.

Extraction

I have Extracted data from on-premise which is in CSV format by using Python that was written and run on Visual Studio Code. and then the CSV format data files from the loadingzone container are extracted using PySpark in Databricks notebooks. then the parquet format data files in the raw container is extracted using PySpark in the Databricks notebook.
Ingestion

I have Ingested data into the Azure Data Lake Storage Gen2 from LoadingZone container into a raw container with the Same hierarchy in CSV format by creating a Python script with all the connection details of the storage container. and then, the CSV data files are taken from the loadingzone and placed into the raw container in the parquet format using the Databricks notebooks to create the compute cluster with the unity catalog. then these parquet files are ingested into the clean container by creating delta tables using the PySpark in the Databricks notebook.
Validation

I have validated the StatusId and AgencyId columns in the products database with a non-numeric regular expression. After validating the columns' integrity, I overwrote the table data in the logosdb catalog's clean container. In addition, I used an email regular expression to validate email addresses in the users table, ensuring that they all followed the correct format. The validated data was then rewritten in the users table of the same clean container in the logosdb catalog.
Transformation

The code removes rows from products table if any of the columns StatusModifiedDate, CreatedDate, or ModifiedDate have missing values. and the general code that eliminates duplicates from all the tables and removes rows if all columns have missing values.
Modeling

here I have assumed the business logic by doing this we can get an in-depth analysis of customer behavior, such as purchasing patterns, frequency, and preferences by Joining user information with order details.

Integrating product information with order details helps evaluate product performance, identify bestsellers, and understand product demand.

Joining Orders and Order Lines: Combines order-level information with line-level detail. It helps in analyzing orders at a more granular level by looking at individual items within each order.

Joining Order Details and Users: Links order details with user information.
Allows analysis of customer behavior, such as order frequency and customer demographics.

Joining Order Details Users and SKUs: Integrates product information with the enriched order details and user data. Enables comprehensive analysis of sales data, including product performance and customer preferences.

Screenshot 2024-05-31 at 1.02.01 AM.png

Screenshot 2024-05-31 at 1.01.03 AM.png

The relationship between Orders and orderlines is Each order can have multiple order lines.

The join resultant of orders and orderlines is joined with the user table we can know each order to the user who placed it. It provides details about the customer associated with each order.

the above resultant is joined with Skus we can know each order line with the specific SKU, giving detailed information about the products ordered.

Semantic Design

Screenshot 2024-05-31 at 1.09.48 AM.png

By joining the Products table and Skus table we can get the Product details associated with each SKU.

The data reveals attributes specific to SKUs, such as SalePrice, RetailPrice, DeliveryMethod, TaxCode, and others. This allows us to analyze pricing, delivery methods, and tax implications for each SKU variation.

We can get access to broader product details like StatusId, AgencyId, Blurb (description), CanSeeInside (availability of product preview), and more. This provides context about the product's lifecycle, categorization, and potential additional details.

Screenshot 2024-05-31 at 1.12.17 AM.png

By joining orderlines and orders tables we can get Complete order line information

When an OrderTXNID matches in both tables, the join combines order line information with relevant order details like OrderDate, UserID, OrderType, TaxAmount, ShippingAmount, and others from the orders_df. This provides a more comprehensive view of fulfilled orders, including the associated line items.

By examining unmatched rows (where OrderTXNID exists in orderlines_df but not in orders_df), we can uncover issues in order processing or data inconsistencies. This can help identify areas for improvement in your order management system.

Operationalization

Error Notifications: Orchestrated the pipeline using Azure Data Factory. added the web activity to send email notifications if the pipeline failure occurs along with the data factory name, and pipeline name using the logic apps by creating the HTTPS request, and sending Email action by passing the Data factory name, pipeline name, and receiver parameters in the body.
CICD: created a sample CI/CD pipeline that helps to push the adf from dev resources to prod resources.
Screenshot 2024-05-31 at 1.56.00 PM.png

A CI artifact is created that helps in pushing the changes from lower environments to higher environments with the help of release jobs or tasks in Azure pipelines.
Here in the CI I deployed the ARM template with both ARMTemplateforfactory.json file for the pipeline code and ARMTemplateparametersforfactory.json for the resources details of the prod environment. and end triggers enable continuous integration which facilitates once the publish happened CI runs automatically.
Note: I used Azure DevOps
