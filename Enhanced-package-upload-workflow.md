Status: **Reviewed**


## Issue
[Enhanced package upload workflow](https://github.com/NuGet/NuGetGallery/issues/4478)

## Problem
Going forward, NuGet packages shall be subject to additional validations that we currently do not perform, and this might result in a slight delay to the publishing process when the package is being validated, indexed and is made available for consumption on the gallery and from the v3/v2 feed. At present, there is no way for the publisher to know if the validation fails, and what to do next.

## Who is the customer?
NuGet package publishers on NuGet.org

## Evidence
The NuGet team receives tens of requests a week specifically asking for clarification around when a package will be available for consumption while it is being indexed. This problem might get exasperated when we introduce package validation as part of this flow.

## Solution

### Modified workflow
Here is the workflow when a package is pushed:

(you may have to zoom-in up to 160%)
![](https://github.com/NuGet/Home/blob/dev/resources/PackageUploadWorkflow/Package%20Upload%20Workflow.png)

### Package page showing the status
This page will be visible only to the package owner/s.

![](https://github.com/NuGet/Home/blob/dev/resources/PackageUploadWorkflow/package%20status%20page.PNG)

### Issues with validation
In the scenario where the package validation fails, the package page updates with the status "package publishing failed". The owners are notified of the same via email, and the NuGet team is alerted as well. At this point, NuGet support will reach out to the owners with the next steps. 

![](https://github.com/NuGet/Home/blob/dev/resources/PackageUploadWorkflow/package%20status%20page%20val%20fail.PNG)

### Post validation and indexing
Once the package has been validated and indexed, the status info bar goes away, the package page becomes public, the package is made available for download from the gallery and for consumption through the v2/v3 feeds, and an email notification is sent to the owners informing them that the package has been published.