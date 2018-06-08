## Problem
1. Privacy concerns - Out of the box, with the default configured source, when a user browses NuGet packages using the package manager UI, VS tries to fetch the icon from the URL provided by the package author. There are privacy concerns with this approach since hitting the icon URL potentially exposes the user’s IP and location.
2. Security concerns -  The location of the image may have been compromised and the image may have been replaced with something malicious. [#2613](https://github.com/NuGet/NuGetGallery/issues/2613)
3. PMUI performance for local feeds - When browsing packages from a local feed on a machine that does not have internet access, the PMUI experiences a UI delay since NuGet tries to fetch icons from the iconurl specified in the nuspec. [#6504](https://github.com/NuGet/Home/issues/6504)


## Who is the customer?
NuGet package consumers who use the NuGet package manager UI.

## Goals
Provide a secured out-of-the-box experience. Packages that come from the default configured source (NuGet.org) or the ones that ship with VS should not leak privacy.

## Solution

### Host icon image on NuGet.org
NuGet.org (herein called "the gallery") would host the icon image for the packages.

On package ingestion, the gallery would read the icon URL from the nuspec, fetch the image and store it. For search, registration metadata, and v2 API (basically any service that customers hit where we expose the icon URL), the icon URL would be replaced with URL to the gallery hosted image location. These are the APIs that are called by Visual Studio when a user is on the "Browse" tab of the package manager UI. No client change would be required for this part. A one time job would be run to fetch, store, and index the images for existing packages.

The user can also upload the icon from the package details page:

![](https://github.com/NuGet/Engineering/blob/master/Staged%20PM%20Specs/Images/IconPrivacy/edit%20package%20icon.PNG)

**Note:**
* Changing the image at the URL provided in the nuspec will not change the icon displayed by the gallery or the PMUI i.e. the gallery will not attempt to fetch the icon for a package that has been published to NuGet.org.
* For sources that do not provide icon URL (such as local sources), the client shall fallback to using the URL from the nuspec.

**PMUI Behavior for installed packages**

When a package is being installed (includes updating to a newer version), the client will fetch and cache the icon image. This cache will be used to display the icon for installed packages. If the icon has not been cached, fall back to using the URL from the nuspec, and then cache the image.

**VS Offline Packages source**

The VS Offline Packages source should come with prepoulated icon cache and the client should utilize that for the browse scenario.

### Icon image as part of the nupkg
There are scenarios where it is not desirable for VS to require an internet connection to fetch images. [#352](https://github.com/NuGet/Home/issues/352) [#6504](https://github.com/NuGet/Home/issues/6504) [#2613](https://github.com/NuGet/NuGetGallery/issues/2613). The go forward strategy would be to allow icon images to be a part of the nupkg.

* the image file should have the same name as the package id and placed at the root of the package
* nuget pack will use an image file placed next to the nuspec which matches the <PackageId>.<ImageExtension> pattern and include it as part of the nupkg


Pros:
* Does't require the client to make additional calls to get the icon, improving perf and especially offline behavior.
* Special importance for Enterprise scenarios where is it desirable for NuGet to not require an internet connection.

Caveats:
* Won't work for existing packages when we stop respecting the icon URLs provided in the package. We cannot repackage the nupkg since that could potentially break the package (filename overlap, targets that enumerates the package files, etc.)

