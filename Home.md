##Welcome to the NuGet team Wiki.

The NuGet Team Wiki will be primarily used for 3 things:-

1. Documenting and updating product roadmap
2. Design Meeting notes
3. NuGet product specifications

###The intent of this area

We will create design discussion summaries here,9 for each design discussion we will open a respective *discussion* issue in the new Discussions Milestone. These issues will stay open for a relatively long amount of time, and are for free discussions. They can turn into actionable issues, or remain in discussion state.

Similarly issues raised by the community that don't yet clearly fall into a "Yes we want to try and do it" bucket, will go to the discussions milestone.

###Why are we doing it
It keeps ideas alive and floating around, and doesn't force us to make decisions like we do when we triage issues into milestones. It keeps actual issues we want to execute on short and to the point.

###Design meeting notes

####[Csproj to Xproj, September 2, 2015](https://github.com/NuGet/Home/wiki/Csproj-to-Xproj-reference-design-meeting-notes-September-2,-2015)

This meeting discussed the design and scenarios for supporting references between Csproj and Xproj projects.

####[UI design, August 21, 2015](https://github.com/NuGet/Home/wiki/NuGet-UI-design-meeting-notes-August-21,-2015)

Follow up meeting to the one on the 20th where we discussed the layout of the list and the middle panel. Next meeting planned for the week of Aug 31st

####[UI design, August 20, 2015](https://github.com/NuGet/Home/wiki/NuGet-UI-design-meeting-notes-August-20-2015)

In this meeting we covered the design for the top bar of the package management dialog.

Please use the design meeting [discussion issue](https://github.com/NuGet/Home/issues/1236) to provide feedback, ask questions, etc.

####[Lock file design, August 18, 2015](https://github.com/NuGet/Home/wiki/Lock-file-design-meeting-notes---August-18,-2015)
In this meeting we covered lock file design, package install and update scenarios for locked projects, and discussed how the current project locking scenarios could be improved by introducing a snapshot file to store only the locked package versions.

Please use the design meeting [discussion issue](https://github.com/NuGet/Home/issues/1233) to provide feedback, ask questions, etc.

####[MsBuild/project.json/xunit Design meeting notes August, 12 2015](https://github.com/NuGet/Home/wiki/MsBuild---project.json---xunit-Design-meeting-notes-August,-12-2015)

Discussing several limitations of project.json + msbuild scenarios. Particularly support for xunit (or other test runners) requiring copy local.

Planning a force copy by Update I, and need further discussion on package spec to perhaps include it as an RID?
