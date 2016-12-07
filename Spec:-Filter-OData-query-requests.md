# Filter OData query requests

## Issue

Feedback on the spec can be given on the following issue - https://github.com/NuGet/NuGetGallery/issues/3407 

## Problem

The NuGet uses an internal Azure SQL db for storing information about package metadata being uploaded. The database is used by the NuGetGallery service to locate package information. One of the SQL performance indicator metrics is DTU, that is a function of memory, compute and disk that the database can consume. The higher the DTU percentage the higher the chances that the query latencies will increase. Currently for the NuGetGallery database have been seen spikes of 80% percentage DTU. The goal is to reduce the DTU for the NuGetGallery database.

## Who is the customer?

All the customers doing search queries for NuGet packages.

## Evidence
Spikes of DTU over a regular week often reach 80% and 90%.


## Solution

The complete solution and investigations will be done over a set of individual tasks. The first task is to stabilize the system by allowing only a set of known query patterns for the OData operators. Other queries that do not respect the defined contracts will be rejected. 
This document refers to this first task.  
The NuGet web apis support the OData protocol. The web requests can include operators as top, skip and orderby that can be expensive while executed against the current SQL design. Using the current requests data captured by Application Insights it is possible to identify the OData operators used by each request. The complete set of data used for filtering is at the bottom of this page.

### _Web APIs_
Filtering will be enabled for the below set of apis.

* ODataV2FeedController.Get **Route**: /api/v2/Packages
* ODataV2FeedController.GetCount **Route**:/api/v2/Packages/$count
* ODataV2FeedController.Search **Route**:/api/v2/Search()
* ODataV2FeedController.SearchCount **Route**:/api/v2/Search()/$count
* ODataV2FeedController.GetUpdates **Route**:/api/v2/GetUpdates()
* ODataV2FeedController. GetUpdatesCount **Route**:/api/v2/GetUpdates()/$count
* ODataV1FeedController.Get **Route**:/api/v1/Packages
* ODataV1FeedController.GetCount **Route**:/api/v1/Packages/$count
* ODataV1FeedController.Search **Route**:/api/v1/Search()
* ODataV1FeedController.SearchCount **Route**:/api/v1/Search()/$count	


### _Data Filters_
The filters for each web api are stored as embedded resources json files. The json schema is described by the ODataQueryRequest type below. The schema can be extended if deeper filtering is needed.
>     public class ODataQueryRequest
>     {
>         public List<string> AllowedOperatorPatterns
>         {
>             get;
>             set;
>         }
>     }

The OData operators will be mapped with the following flags:
>         [Flags]
>         public enum ODataOperators
>         {
>             none = 0,
>             expand = 1,
>             filter = 1 << 1,
>             format = 1 << 2,
>             inlinecount = 1 << 3,
>             orderby = 1 << 4,
>             select = 1 << 5,
>             skip = 1 << 6,
>             skiptoken = 1 << 7,
>             top = 1 << 8
>         }

### _Filtering_
The filtering is done at each web api level by comparing the received ODataOptions for the request against the allowed query patterns for the current api.
Return result is System.Web.Http.Results.BadRequestErrorMessageResult for the cased when a query is filtered out.
The requests not using OData operators will not be filtered out. 

### _Exception Handling_
All the exceptions are handled. In case of an exception:
1. The information is logged in the Application Insights.
1. The execution continues without filtering.

### _Opt In / Opt Out_
Enabling or disabling the feature is possible by Web config setting. The setting will be available through IAppConfiguration.
Default Value: false

`<add key="Gallery.ODataFilterEnabled" value="false" />`

### _Dynamic filter pattern generation_
Out of the scope. 

### _Updates_
If new patterns need to be used the service needs to be redeployed with the new query patterns.

### _Supported Query Patterns_

* _**/api/v2/Packages and /api/v2/Packages/$count**_

 > ` "AllowedOperatorPatterns": [`
>     `"filter",`
>     `"filter, orderby, top",`
>     `"filter, top",`
>     `"filter, orderby, skip",`
>     `"filter, orderby",`
>     `"filter, orderby, select, top",`
>     `"filter, skip",`
>     `"filter, orderby, skip, top",`
>     `"skip",`
>     `"filter, format, orderby, select, skip, top",`
>     `"filter, select",`
>     `"orderby, top",`
>     `"filter, inlinecount, orderby, skip, top",`
>     `"filter, orderby, skiptoken, top",`
>     `"filter, format, select",`
>     `"orderby, skip",`
>     `"filter, skip, top",`
>     `"filter, inlinecount, orderby, select, skip, top",`
>     `"filter, orderby, select, skip, top",`
>     `"filter, orderby, select",`
>     `"filter, select, skip, top",`
>     `"filter, format, select, top",`
>     `"orderby, skip, top",`
>     `"top",`
>     `"filter, orderby, select, skip",`
>     `"orderby",`
>     `"skip, top",`
>     `"filter, select, top",`
>     `"filter, inlinecount, select, top"] `

* _**/api/v2/Search()? and /api/v2/Search()?/$count**_

 > ` "AllowedOperatorPatterns": [`
>     `"filter, skip, top",`
>     `"filter, orderby, skip, top",`
>     `"top",`
>     `"filter, orderby",`
>     `"filter, orderby, skip",`
>     `"filter, top",`
>     `"filter, inlinecount, orderby, skip, top",`
>     `"orderby, skip, top",`
>     `"skip, top",`
>     `"filter, orderby, top",`
>     `"filter, inlinecount, orderby, select, skip, top",`
>     `"orderby, top"`
>   `]`

* _**/api/v2/GetUpdates()? and /api/v2/GetUpdates()?/$count**_

  > `"AllowedOperatorPatterns": [`
>     `"orderby, top",`
>     `"filter, orderby, top",`
>     `"filter",`
>     `"orderby, skip, top",`
>     `"filter, skip, top",`
>     `"skip",`
>     `"skiptoken",`
>     `"filter, orderby, skip, top"`
>   `]`

* _**/api/v1/Packages and /api/v1/Packages/$count**_

 > `"AllowedOperatorPatterns": [`
>     `"filter, orderby, top",`
>     `"filter, top",`
>     `"filter, orderby, skip, top",`
>     `"filter, select",`
>     `"top",`
>     `"filter, orderby"`
>   `] `

* _**/api/v1/Search()? and /api/v1/Search()?/$count**_

 > ` "AllowedOperatorPatterns": [`
>     `"filter, orderby, skip, top",`
>     `"filter, skip, top",`
>     `"filter, top" `
>   `]`
