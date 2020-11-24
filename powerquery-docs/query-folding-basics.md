---
title: Query folding basics
description: Understanding the basic principles of query folding to get the most out of your Power Query experience and optimize your queries.
author: ptyx507
ms.service: powerquery
ms.reviewer: 
ms.date: 09/02/2020
ms.author: v-miesco
---
# Query folding basics

Whenever you apply transforms to source data in Power Query, it does its best to have as many as possible of these performed on the data source, rather than locally (on your machine or in the cloud service, depending). This is called "Query Folding". All of the transforms you apply when working in Power Query are stored in a document (which can be viewed in the Advanced Editor) written in the "M language"(link to M function reference here), and a subset of them are turned into the native query language (such as SQL, API calls, etc.) of your data source.

Depending on how the query is structured, there could be three (3) possible outcomes for this mechanism:
* **Full Query Folding** - When all of your query transformations get pushed back to the data source and no processing occurs locally by the Power Query engine. Instead you receive your desired output directly from the data source.
* **Partial Query Folding** - When only a few transformations in your query, and not all, can be pushed back to the data source. This means that a subset of your transformations will be performed at your data source and the rest of your query transformations will occur locally.
* **No Query folding** -  When the query contains transformations that can't be translated to the native query language of your data source either because the transformations are not supported or the connector doesn't support Query folding. For this case Power Query gets the raw data from your data source and works locally with the Power Query engine to achieve your desired output.

>[!NOTE]
>The Query folding mechanism is primarily available in connectors for structured data sources such as but not limited to [Microsoft SQL Server](Connectors/sqlserver.md) and [OData Feed](Connectors/odatafeed.md). 
>
>Leveraging a data source that has more processing resources and has Query folding capabilities can expedite your query loading times as the prorcessing will occur at the data source and not locally in the Power Query engine.

This article will try to provide some example scenarios for each of the possible outcomes for query folding in the next sections as well as some suggestions on how to get the most out of the Query folding mechanism.

## Full Query folding

For this scenario, the data that you will be connecting to is Microsoft SQL Server and the sample database is the AdventureWorks in its Data Warehouse version, which you can download from the article [AdventureWorks sample database](https://docs.microsoft.com/sql/samples/adventureworks-install-configure).

After identifying the data source, it is suggested that you pick the native connectors found in the 'Get Data' window. In this case, the connector to be used is the [Microsoft SQL Server Connector](Connectors/SQLServer.md).

Your goal is to summarize the data inside the *FactInternetSales* table by performing the following transformations:

* Only get the data from September 2012 by filtering the rows on the *OrderDate* column

![Filtering the FactInternetSales table by the OrderDate column for only dates in the month of September 2012](images/me-query-folding-basics-filter-values.png)

>[!NOTE]
> You can read more on how to filter rows by their values from the article [Filter values](filter-values.md).

* Group by the *OrderDate* column and create a new aggregated column using the Sum operation on the *SalesAmount* column. Name this new column *Total Sales Amount*.

![Group by using the OrderDate column and aggregating by the SalesAmount column](images/me-query-folding-basics-group-by.png)

>[!NOTE]
> You can read more on how to use the Group by feature from the article [Grouping or summarizing rows](group-by.md).

* Now with the summarized table at the date level, filter the new *Total Sales Amount* column to only keep rows with values greater than or equal to 15000.

![Filtering the summarized table by the Total Sales Amount column for values greater than or equal to 15000](images/me-query-folding-basics-filter-values-greater-than.png)

One simple way to check if the step in your query can fold back to the data source is to right-click the step and see if the **View Native Query** option is enabled out or disabled / greyed.

![Right-clicking the last step of the query to check the View Native Query option](images/me-query-folding-basics-view-native-query.png)

When you click the **View Native Query** option, a new window will appear called **Native Query** where you'll see the Native Query that Power Query has translated from all the transformations that construct the selected step.

![The Native Query window with the SQL code generated by Power Query](images/me-query-folding-basics-native-query-window.png)

This native query will be sent to the data source (Microsoft SQL Server) and Power Query will only receive the result of that query.

## Partial Query folding

Taking the query created in the previous section for **Full Query folding** as your starting point, your new goal is to filter that table to only analyze the rows for dates that fall in the weekdays Friday, Saturday, or Sunday.

To do this, first select the **OrderDate** column. In the **Add Column** menu from ribbon, select the *Date* option found in the *From Date & Time* group, which will display a new menu. From this new submenu, select the option that reads *Day*, and this will display a new submenu where you can select the *Name of Day* option. 

![Option to add a new column for the Name of the Day](images/me-query-folding-basics-weekday-name.png)

After clicking this button, a new column called **Day name** will appear in your table with the name of the Weekday. You can now filter the table using this **Day name** column to only keep the rows with the values Friday, Saturday, or Sunday.

![Filtering the summarized table using the name of the Day](images/me-query-folding-basics-filter-weekday-name.png)

You can now check the **View Native Query** option for the last step that you created. You will notice that this option will appear greyed out or disabled. However, you can right-click the *Filtered Rows1* step and you'll see that the **View Native Query** option is available for that step.

For this particular scenario, the query will be partially folded to the data source until the *Filtered Rows1* step.

![View Native query disabled after performing new transformations](images/me-query-folding-basics-disabled-view-native-query.png)

Another option to verify query folding is to use the Query diagnostics tools, more specifically the **Diagnose Step** option. You can learn more how to use the Query diagnostics tool from the article on *[What is Query Diagnostics for Power Query?](querydiagnostics.md)*.

![Query diagnostics tools in the Power Query ribbon](images/me-query-folding-basics-query-diagnostics.png)

To use it, you simply need to select the query that you want to analyze and then hit the **Diagnose Step** button. This will create a new group and two queries with the format "[Query Name] [Step Name] [Diagnostics Type] [Timestamp]".

You can take a look at the one that reads "Aggregated" in the [Diagnostics Type] part and take a closer look at the field with the name Data Source Query. This column holds all the requests sent to the data source.

![Query diagnostics at the step level for the last step of the new query showing the requests sent to the data source in the Data Source Query field](images/me-query-folding-basics-query-diagnostics-aggregated-view.png)

Reading the values in that column, you can see that the native query sent to the server to retrieve the information. You can right-click to drill down to a specific value and see the contents of the cell specifically value in row twenty first in the previous image that is the same native query that you can see in the **View Native Query** for the **Filtered Rows1** step.

![Value found inside the query for the aggregated query diagnostics which holds the SQL statement sent to the SQL Server](images/me-query-folding-basics-query-diagnostics-aggregated-view-drill-down.png)

This means that your query will send that native query to the Microsoft SQL Server and perform the rest of the transformations locally. This is what it means to have a query that can partially fold.

>[!NOTE]
> It is highly recommended that you read the article on [Understanding folding with Query Diagnostics](querydiagnosticsfolding.md) to get the most out of the Query Diagnostics tools and learn how to verify query folding.

## No Query folding

Queries that rely solely on unstructured data sources such as CSV, or Excel files do not have Query folding capabilities. This means that Power Query will evaluate all the required data transformations outside the data source.

One example can be seen in the article on [combining multiple CSV files from a local folder](combine-files-csv.md) where none of the steps have the *View Native Query* active and running the Query diagnostics for that step yield no results in the *Data Source Query* field.

![View Native Query greyed out for the query that combine CSV files](images/me-query-folding-basics-csv-files-source.png)

## Considerations and suggestions

* Follow the best practices when creating a new query as stated in the article on [Best practices in Power Query](best-practices.md)
* Checking the **View Native Query** option is always recommended to make sure that your query can be folded back to the data source. If your step disables this option, you know that you've created a step that stops query folding. 
* Use the Query diagnostics tool to your advantage and to better understand the requests being sent to your data source when query folding capabilities are available for the connector.
* When combining data sourced from the usage of multiple connectors, Power Query will try to push as much work as possible to both of the data sources while complying with the privacy levels defined for each data source. 
* Read the article on [Privacy levels](dataprivacyfirewall.md) to protect your queries from running against a Data Privacy Firewall error.
* You can also use other tools to check query folding from the perspective of the request being received by the data source. Based on our example, you can use the Microsoft SQL Server Profile to check the requests being sent by Power Query and received by the Microsoft SQL Server. 
