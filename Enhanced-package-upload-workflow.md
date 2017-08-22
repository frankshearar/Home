## Issue
[Enhanced package upload workflow](https://github.com/NuGet/Home/issues/5662)

## Problem
Going forward, NuGet packages shall undergo validation, in addition to indexing, before they are made available on the gallery and for download from v3/v2 feed. At present, there is no way for the publisher to know if the validation fails. what is the status, how much time will it take, or if there is a problem with the package.

## Who is the customer?
NuGet package publishers would have an intuitive, self-serve way of knowing the current status of the package and would not have to guess and/or reach out to NuGet support.
Fewer tickets for the NuGet team which means we can spend more time doing awesome things.

## Evidence
The NuGet team receives tens of requests a week specifically asking for clarification around when a package will be available for consumption while it is being indexed. This problem might get exasperated when we introduce package validation as part of this flow.

## Solution
