## Requirements
* When one or more NuGet services are degraded,
  * Auto update the status page
  * Auto update the NuGet.org page with the status message with link to status page
  * Should not refresh and remove the degraded status

* Show the package submission time on the status page
  * Package validation time
  * Indexing time

## Proposal

### Services that should be monitored for auto update of the status page

### NuGet Gallery notifications

### Status page
* Collapse service categories by default. Expand if one of the components/services in the category is degraded.
* V2 and V3 endpoints cleanup in the Service section
* CDN status
* Improve the status messaging - should not auto refresh to Green.

### Status page - Package submission delay graph



