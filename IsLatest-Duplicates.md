#IsLatest Duplicates

##Issue

see [NuGet/NuGetGallery/#2514](https://github.com/NuGet/NuGetGallery/issues/2514)

##Problem

Concurrent CUD operations for two package versions in the same registration can result in duplicate or invalid versions flagged as IsLatest/IsLatestStable.

###Issues:

1. IsLatest/IsLatestStable flags are on the Packages table, not PackageRegistrations
  
  Duplicates are possible because state is stored across multiple rows. This may happen when multiple UpdateIsLatest executions are triggered for concurrent CUD operations for the same package registration.
  
2. UpdateIsLatest has concurrency issues

  Duplicates (above) and/or incorrect IsLatest/IsLatestStable versions are possible due to the following factors:
  * Calculation (UpdateIsLatest) happens from the client(s)/web server(s) rather than a central location
    * Concurrency across multiple threads, same server
	* Concurrency across multiple servers
  * There is no exclusive lock of IsLatest/IsLatestStable calculation per registration
  
##Who is the customer?

Anyone doing concurrent CUD operations on the same package registration, including:
* Package uploads
* Package updates: un/list or (soft) delete
* Package (hard) deletes

##Evidence

* Report from customer

##Solution (Options)

###1. Move IsLatest/IsLatestStable columns to the PackageRegistrations table

Implementation steps (staged rollout):
* Add columns via Gallery EF migrations
* (Optional) Start populating columns for new CUD operations
* One-time job to populate columns for any packages not yet updated
* Change UpdateIsLatest in Gallery to use new columns

Pro: Solves duplication issue<br/>
Con: Does not solve concurrency issue, requires schema change and staged rollout

###2. Enable Optimistic Concurrency

Implementation details:
* Add RowVersion column to Packages table OR
* Add ConcurrencyCheck attribute to IsLatest/IsLatestStable columns in EF model
* Handle new DbUpdateConcurrencyExceptions with retries from Packages and Api controllers

Pro: Solves concurrency issue, simple code change, no DB changes<br/>
Con: Can break existing scenarios unless new exceptions are handled with retries

see: [https://github.com/NuGet/NuGetGallery/commit/2b8d0c91d00f9c222b4ddc998dd3facd1c1791ee](2b8d0c9) (Still missing exception handling and retries)

###3. Move IsLatest/IsLatestStable calculation (PackageService::UpdateIsLatestAsync) into the DB

Implementation details:
* Port UpdateIsLatest to TSQL stored procedure
* Add triggers, along with exclusive lock on all Packages rows for the registration
* Remove UpdateIsLatest from Gallery

Pro: Solves concurrency issue, by moving calculation to central location with exclusive lock<br/>
Con: Complicated TSQL code, need new project for DB tests, Gallery needs to re-select to get updates, need perf/stress testing

see: [https://github.com/NuGet/NuGetGallery/commit/57e7eaa54e116fa05abd1e9766f3fae3b44a4970](57e7eaa) (Still missing re-select, has bug with triggers)

###4. Move UpdateIsLatest calculation to background worker

Implementation details:
* Move UpdateIsLatest code from Gallery to background job (single instance)
* Add exclusive per-registration lock
* Queue background job processing from Gallery

Pro: Solves concurrency issue, by moving calculation to central location with exclusive lock<br/>
Con: Adds complexity with new Job and Gallery queuing mechanism

###Examples:  
```
1. Concurrent Uploads: duplicates
[UPLOAD-1]: SELECT; pkgs=(alpha1)
[UPLOAD-1]: INSERT alpha2; pkgs=(alpha1, alpha2)
[UPLOAD-2]: SELECT; pkgs=(alpha1)
[UPLOAD-2]: INSERT alpha3; pkgs=(alpha1, alpha3)
[UPLOAD-1]: Clear IsLatest/IsLatestStable
[UPLOAD-2]: Clear IsLatest/IsLatestStable
[UPLOAD-2]: UpdateIsLatest=alpha3
[UPLOAD-1]: UpdateIsLatest=alpha2

2. Concurrent Uploads: wrong version
[UPLOAD-1]: SELECT; pkgs=(alpha1)
[UPLOAD-1]: INSERT alpha2; pkgs=(alpha1, alpha2)
[UPLOAD-2]: SELECT; pkgs=(alpha1)
[UPLOAD-2]: INSERT alpha3; pkgs=(alpha1, alpha3)
[UPLOAD-2]: Clear IsLatest/IsLatestStable
[UPLOAD-2]: UpdateIsLatest=alpha3
[UPLOAD-1]: Clear IsLatest/IsLatestStable
[UPLOAD-1]: UpdateIsLatest=alpha2
```