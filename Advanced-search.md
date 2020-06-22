# Background
Package search is a key feature on NuGet.org. Many customers asked for sorting and filtering features for the NuGet search feature [Customer feedback](#Customer-feedback). The sorting feature is already available for advanced users and the goal of this project is to make sorting and filtering features conveniently accessible from the NuGet.org UI.

## Experience Today
Currently, it's really difficult for users to  filter search results by package type. They would have to know that the feature exists and then make a manual request to the public API with the correct parameter name and value. 

Advanced users can sort packages by many criteria, including:
* relevance
* lastEdited
* published
* title-asc
* title-desc
* created-asc
* created-desc

As of now, there are multiple different package types and it is difficult for users to filter their search results depending on the package type. For example, it is difficult to find .NET tools and .NET templates in NuGet. In fact, people manually curated lists of .NET tools: https://github.com/natemcmaster/dotnet-tools


As for packages relevance, the current search algorithm doesn't work well if the best result's package ID is very different from common searches. For example, a search for "barcode" doesn't put the "zxing.*" packages at the top even if they are the most downloaded packages for that usage.

# Goals
* Create a pleasant user experience to access sorting/filtering features from NuGet.org 
	
# Non-Goals
* Improve search result relevance (scoring algorithm)
* Modify the NuGet protocol
* Including the Visual Studio UI in the intern MVP
* Implementing the Target Framework Moniker (TFM) filtering
* Change the V3 API endpoint

# Solution

The solution consists of applying changes to 2 parts: Search service and Gallery

## Gallery
For the Gallery side, a new UI will be added to the search result page to be able to access the advanced search features. Here are few mockups of possible solutions:

### Mockup
#### Collapsed
![Collapsed search panel](https://user-images.githubusercontent.com/65630625/85337567-bc9df880-b4ae-11ea-8a0d-f3fc66244b7b.png)

#### Expanded
![Expanded search panel](https://user-images.githubusercontent.com/65630625/85337622-d3dce600-b4ae-11ea-8065-5da4a5a828ec.png)

#### Mobile view
![Mobile view](https://user-images.githubusercontent.com/65630625/85337737-fff86700-b4ae-11ea-8282-60f39a661ec6.png)


## How we chose the "Sort by" and "Package type" parameters values
Many of the selected options for sorting were chosen because our current search infrastructure already supports them or allows us to easily add them without impacting the current existing features. For example, the "Downloads" sort option will sort the search results by the total download count of the package (in descending order). As for the "Recently updated" it would sort packages by their "latest version's creation date".

As for the package type filtering, since the field is customizable by the customer, we have many different package types that have extremely low usage.

Thus, we looked at the current count of packages in  each category and included the categories that contain the majority of our hosted packages. Today, 99.3% of packages are regular .NET dependencies. If we exclude dependencies, .NET Tools are 49% of the remaining packages, and .NET Templates are 30% of the remaining packages. Other package types make up a significantly smaller portion of our packages (single digits %).

# Measuring Success

These are the current KPIs (Key Performance Indicators) and how we think they may be affected:
* Bounce rate (May increase if we don't exclude advanced search from the results)
* Average click index on first page (May improve since advanced search should bring the desired results close to the top)
* Clicks on first page % (May improve since sorting & filtering brings the most wanted results close to the first page)

There are already some KPIs in place for the current search. The goal of this project is to not affect them negatively.

It is hard to measure success for the newly added advanced search. Thus, we will mainly look at customer behaviour and measure adoption using the collected telemetry.  For example, today we see that 0.8% of searches include the "hide prerelease" filter and we expect that number to not get lower.

We will also ask for customer feedback from the NuGet authors DL and Twitter communities.

# Future Work
Adding those features to NuGet.org website will allow us to collect telemetry and measure the success of the newly added feature. The learning acquired with the release of the feature will help us shape a better experience for our customers as well as design a pleasant experience for our Visual Studio users.

Adding more advanced searching criteria and filtering by specific fields are also features we are considering implementing to help our customers have a more customized experience.

We also think that filtering packages by Target Framework Moniker (TFM) would be a useful addition that will help   customers in their usage of our platform as well as help us in the future for implementing features such as package compatibility search (searching for packages that are compatible with our project).

# Customer feeback

**TODO**
