In this meeting we covered the flow where a user wants to update/install/uninstall a package. Today there are several issues

1. The right side selector for projects is confusing. And users are treating it incorrectly because of muscle memory. In NuGet 2.x unchecking a project meant removing the package from the project. And many users still think that's the case. This is particularly problematic for developers that have both versions side by side. We want to come up with a mechanism that makes this clear.
2. The V2 extensions provided actions directly on the list, leading to less wandering around on the page, and less clicks.
3. The dropdown selecting the action, and then clicking on the install button (or others) like in the image below, is a really confusing UI design pattern. It also requires multiple unnecessary clicks. We want to simplify the flow and make the flow extremely clear.

Existing UI:
![image](https://cloud.githubusercontent.com/assets/1238711/9447468/59d67136-4a4c-11e5-9a71-5f62c1083713.png)

We tried separating the actions to individual tabs, or individual buttons. But the result seemed too cumbersome and took too much real estate and didn't really achieve the goal of resolving issue 3.

The direction we are currently exploring is adding buttons on the list (where icons exist today). The image below shows the buttons (but not how we want them to look like). The actual available buttons are going to get filtered, based on available actions.

We imagine the right hand side of the screen to show what projects already include the package and in what version.

We want to restore the tree view (rather than a flat list) of the projects.

Once a button is clicked, a preview screen comes up, that lets the user select what projects to apply the action to. The gesture on this UI should be different than a regular checkbox to prevent the muscle memory issue discussed in item 1. We imagine that when uninstalling, the projects that are going to get removed are going to be marked with an icon signifying delete (with tooltips), and similarly for install/update.

Suggeted UI:
![image](https://cloud.githubusercontent.com/assets/1238711/9447328/a956d8dc-4a4b-11e5-8c6a-4920ab6c28ee.png)