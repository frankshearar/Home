###project.json + MSBuild support currently has a few broken scenarios (as of WinTool 1 & NuGet 3.1.1)
 
Participants: Jason Malinowski, Justin Emgarten, David Fowler, Kevin Pilch Bison, Yishai Galatzer

1. Adding a xUnit package to a class library doesn’t know that we really need to copy the package, despite it being a class library.
2. Adding the EF tooling package and others expect DLLs from their packages are copied into the output directory so they can load them for tooling.
 
There are potentially several pieces of work needed. 
 
1. The build task needs to copy stuff into class libraries, but we don’t always want to do that for true class libraries. Some fancier logic might be needed.
2. You probably don’t want these “tooling” packages to flow between projects (or do you?), and definitely don’t want them to be packaged in your app. Some additional control might be needed.
3. Right now people putting stuff in lib are getting stuff for the target framework as opposed to something that will run on your host OS. This doesn’t work in cross-platform cases, so some better packaging spec might be needed.
 
Notes
 
Console app with AnyCPU should fail if the packages can't apply.
But we have no way to know whether it's optional or mandatory.
Bug to file: NuGet should error out unless you have an RID that is applicable.

+ Could we have DNX run the runners to do all the loading magic?
+    For the DesignTime hack, there's already a solution
+    TODO: tell them that the feature exists
+ How do people actually do new top-level folders? (Like Analyzers)
+    A reservation system is needed (Let the NuGet team know by sending a PR into nugetdocs repo)
+	TODO: tell Jon to merge our analyzer spec into the master spec (Done)
+ We could just have some option in the class library or PCL projects to specify a "special" RID that should be copied regardless. (Yes)
+    Definite for Update 1: change to always copy for class library

####Next day followup

1. For the suggestion that NuGet should fail a restore if there’s no applicable native/runtime asset for your RID, filed (bug 1173)[https://github.com/nuget/home/issues/1173). I don’t think it’s a “need” right now, so NuGet folks feel free to triage as needed (moved to 3.3)
1. For the suggestion that we should always copy assets for desktop-targeting class libraries, filed DevDiv bug gist below.
1. analyzer spec to be added to nugetdocs (done)

####Gist of DevDiv bug:
 
1. Create a new desktop class library.
2. Manually add a project.json, and add a reference to the xunit test binaries.
3. Build.

Expected: the xunit test assets are copied alongside the test binary
Actual: they aren't