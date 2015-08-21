# UI Design meeting notes

We have gotten feedback about the NuGet UI, and decided to address it in two steps.

Step 1. Perform relatively small/manageable changes to the UI to address the bulk of the feedback, but not necessarily overhaul it or add new features
Step 2. Perform another design iteration to address further feedback, and start incorporating feature requests.

Given the amount of user scenarios, and the variety of user scenarios this will be a series of design discussions where we are going to gradually refine the design.

In this meeting we focused on the top bar, and we started discussing the body of the dialog. We will post the results of that in another design meeting summary.

###Top bar
In the first meeting we focused on the top bar, this is what it looks like in NuGet 3.1.1

![image](https://cloud.githubusercontent.com/assets/1238711/9420470/e3579c98-481a-11e5-9365-4e00bfa674e6.png)

The feedback revolved around the fact that the dropdown filter was confusing, search was forced and was churning out requests, some people wanted to go straight to their installed packages.

We came up with the following sketch as a starting point for a more detailed design

![image](https://cloud.githubusercontent.com/assets/1238711/9420463/cb253e82-481a-11e5-8e1c-204bb2fd7d85.png)

1. The drop down was replaced with a tab like text on top. This simplifies understanding what is it the user is viewing
1. The search was moved to a more central position, and became bigger. Search is an action a user does a lot in the Browse mode.
1. The search box now will stay in place as the dialog resizes, and will not jump to the second line.
1. All was changed to be named Browse. This was done to reduce confusion with All Sources, and to make it clear this is for finding packages, rather than looking at installed packages. The names are not final.
1. Include Prerelease checkbox is now right of the search text box, as it is a filter on search (well mostly)
1. Package source was moved to the top right, since it is not very often the thing to change. Particularly as we want to introduce back the support for "All sources".

