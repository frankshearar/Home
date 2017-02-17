# Issue

[Improve NuGet Gallery DTU](https://github.com/NuGet/NuGetGallery/issues/3280)

# Problem

The NuGetGallery database has DTU spikes that happens almost regularly on daily basis around same time (00:00 UTC until 06:00 UTC). The goal is to understand the reason of these spikes, find/evaluate different solutions and implement the chosen approach to alleviate this behavior.
The current document is a recording of the investigations done, findings and proposed solutions.

# Who is the customer?

NuGet clients that use OData queries to search NuGet data.


# Solution

## # Part1: Cause of DTU spikes
To find a solution for the problem, an understanding of the cause of the problem is needed. This first part is focused on determining an answer for what could determine the DTU spikes. 
NuGet Gallery db can be queried by using the NuGet Gallery OData apis, by using direct query execution through .Net or TSQL. 
The OData apis can be used by a wide set of users through NuGet Gallery endpoints. Direct SQL query can be done only through machines that were given explicit permissions.
Using the DB SQL audits for a specific day it was observed that:
* Most the requests are through OData requests.
* There are a not insignificant number of requests coming from IPs not related with the NuGet gallery web server.These IPs were identified as being used by our internal jobs to run monitoring, stats collect etc.

### Internal Jobs 
Hypothesis: The internal jobs execution is one of the major contributing factor at the DTU spikes.
The internal jobs were studied from the point of execution count and execution interval. 
Details about how data was collected here.
Using the SQL Audit data for the IPs related with jobs’ execution were studied:
* The most expensive queries performed by the jobs
* The distribution of the number of jobs’ queries over the hours.
* The distribution of server duration for these queries.

The three pictures reflect the data collected. 

![NuGet jobs – most expensive queries]
(https://cloud.githubusercontent.com/assets/16580006/23052787/ef1a96c2-f488-11e6-835d-61f7e0424bcb.png)
_NuGet jobs – most expensive queries_



![NuGet jobs – distribution per server duration and hour for SQL queries ](https://cloud.githubusercontent.com/assets/16580006/23055609/9b880a4a-f49b-11e6-847d-18d9de6161a8.png)
_NuGet jobs – distribution per server duration and hour for SQL queries_



![](https://cloud.githubusercontent.com/assets/16580006/23055669/e7fd3012-f49b-11e6-8778-f89c80ca1ccf.png)
_NuGet jobs: distribution per count and hour for SQL queries_



### Conclusions
1. During the spike hours, the SQL jobs do not seem to bring a substantial load on the Server.
1. The further query profiling will confirm this fact.
1. There is room for improvement in the following aspects:
* The daily and half day jobs can be scheduled to run at different hours.
* Investigate the most time consuming queries and possible create new indexes that will improve query execution. For example [dbo].{Packages].{NormalizedVersion] is not indexed. 
* Clean up some of the jobs if they are not used.

## NuGet Gallery
Application Insights 
All the requests from NuGet Gallery an OData end points are recorded in Application Insights. While this data may not be direct reflection on the SQL, statistics based on this data it can help to indicate type of requests that could be related with the DTU spikes.
Using the AI data were executed two experiments.
1. Based on count of requests per hour. 
1. Based on the duration of the requests per hour.
The goal was to determine execution patterns based on Client IP and/or requests’ structure.

**Research using the execution count**
The method for using the count proved to not be successful because of few reasons.
* The request count did not show any increase during the studied time interval (00:00 – 06:00).
![](https://cloud.githubusercontent.com/assets/16580006/23055778/8180d02c-f49c-11e6-9fe5-c5b59964d32a.png)

* Drilling down more on the requests many of them were Not reaching the data base and just used the Search service to search data (PackageById). 

**Research using the request response time**
The hourly graph for request duration shows an increased activity in the hours between 00:00 and 05:00. 
![](https://cloud.githubusercontent.com/assets/16580006/23055835/da3dd034-f49c-11e6-8a27-8a2ae7191eb9.png)
_Hourly distribution for requests based on duration of executions_

Next step is to identify what queries are producing the largest request duration and also are consistent executed daily.
A join of the most 5 costly requests over 6 days did lead two IPs:
![](https://cloud.githubusercontent.com/assets/16580006/23055859/06ba9868-f49d-11e6-9c01-34cd97f55f4d.png)

The study of these two IPs lead to the below facts:
* These IPs were executing four type of query patterns
    1. Packages()/$count?$filter=(LastUpdated gt datetime'2016-11-19T00:35:17.067') and (Published gt datetime'1900-01-01T00:00:00')&$orderby=LastUpdated
    1. Packages()?$filter=(LastUpdated gt datetime'2016-11-19T00:35:17.067') and (Published gt datetime'1900-01-01T00:00:00')&$orderby=LastUpdated
    1. Packages()?$filter=(LastUpdated gt datetime'2016-11-19T00:35:17.067') and (Published gt datetime'1900-01-01T00:00:00')&$orderby=LastUpdated&$skip=100
    1. Packages?$orderby=LastUpdated&$skip=

* The hourly distribution of these two IPs showed that one IPs is active only during the DTU pick hours while the other is working day long.
![](https://cloud.githubusercontent.com/assets/16580006/23055997/ed13de50-f49d-11e6-9d56-cc4dd16c1761.png)

### Conclusions
Based in the App Insights data:
* Two IPs could be the major contributors to the DTU spikes.
* There are 4 types of requests that need to be investigated further.
* Using the “request count” feature was not highly successful in the current investigations.

## SQL 
### Query Store and SQL Profiler
With the previous observations, there are now 4 types of requests that need to be investigated. However, we need to be aware by the following:
These queries were obtained by using the “request response time”. This is only a hint for the problem trying to be solved. There is not any data that shows:
* These queries do SQL querying.
* These queries (even if reach SQL) are CPU intensive. (The high DTU spikes is due to high CPU time).
* Can be more SQL queries that did not come through OData or could not be found through the above approach.

Additional investigations were needed to SQL usage and find a way to map the SQL information with the OData request types.
Query Store  is a tool for profiling SQL queries is enabled by default for Azure V12 data bases.  It offers views and tooling for getting the poorest performing queries based on a set of metrics (CPU, duration etc).
Using the Query Store (using CPU total and CPU average) were found the 10 most consuming SQL queries related to the total CPU and another 10 most consuming queries using the CPU average. The text for the sql queries are quite long and it is added in here. 
Since the NuGet Gallery is using Entity Framework the queries recorded on the SQL are transformed and most of the times only slightly resemblance the OData requests. 
To map SQL query text with the actual request, it was used the SQL Profiler over an OnPrem NuGetGallery db set for a local NuGet web server.
From the 20 high consuming CPU queries 17 were mapped with OData requests. One query did not come from the Gallery. 2 could not be mapped.
The mapping is below.
![](https://cloud.githubusercontent.com/assets/16580006/23056084/5d600efe-f49e-11e6-9f33-5f3ab73943a8.PNG)

**Queries with multiple plans**
When a query is executed sql optimizer calculates an optimal query plan based on indexes and statistics. Different query plans that are considered not being optimal are cached (for an amount of time) and new query plans are created. SQL Store displays the different query plans and let the users force an execution plan if they find it more suitable. This will mean that on the next query execution that query plan will used. Currently only one of the top high CPU queries has more than only plan (has 3) 21868745.

### Conclusions 
* Our internal jobs could be improved to reduce the stress on the NuGet gallery db but they are not the major fact in the DTU spikes.  
    1. Three jobs were can have their execution time slightly changed. 
    1. Dashboard related jobs need to be evaluated to understand what is needed and what is not. 
    1. Evaluate adding more indexes ([dbo].{Packages].{NormalizedVersion)

* AppInsights data proved useful in the current investigation but was not enough in understanding the SQL performance issues. We can improve more our AI pipeline by adding telemetry to reflect the efficiency of the Search service usage. Two OData request types, that could be part of the DTU spikes, were identified 
* SQL Auditing is great tool to detect unexpected clients that use the database. We need to look more into how we can integrate them in our monitoring pipelines. Dashboard jobs were identified through this method.
* Query Store + App Insights + SQL Profiler – represents a good set of tools to find and correlate non-performant sql queries with their web requests. The queries to be investigated are here.

## Next Steps
### Clean Up and Infrastructure
1. Cleanup internal jobs (disable what is not needed). Dashboard jobs in special.
1. Evaluate the possibility to change the execution hour for a set of jobs. Jobs mentioned here.
1. Improve the queries executed as part of the internal jobs. For example [dbo].{Packages].{NormalizedVersion] is not indexed. 
1. Rebuild the data base indexes.
1. Enable Auto_Statistics and Auto_Statistics updates.

### Monitoring
1. Add instrumentation to the source code to indicate how many of the request are going against db vs the Search service.

### Design Changes
1. Investigate the possibility of redirecting more query patterns to run against the Search service instead of against the data base. The search patterns are on the OData APIs but also, they are executed on the Packages controller for the package display and package download.
1. Evaluate the possibility of moving the queries not used by the valued clients (nuget.exe) to a different database. The final solution for this will require having a way of identification for the “valued” clients.
1. The Entity Framework does by default an Order By (primary key) in the current case is the query Id. This Order By may not be necessary in cases when the OData already specifies the OrderBy clause. Research if it is possible to override the EF order by in such cases.
1. Evaluate the possibility of adding an indexed view.
1. Evaluate the possibility of blocking certain columns in OrderBy cases.

### SQL Query improvements
1. Deeper investigations for query performance for each of the queries identified.
1. Reach to the customers that own the low performance queries and help them to improve them.

## References
[https://azure.microsoft.com/en-us/blog/query-store-a-flight-data-recorder-for-your-database/](https://azure.microsoft.com/en-us/blog/query-store-a-flight-data-recorder-for-your-database/)


