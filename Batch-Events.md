# Problem
While updating multiple packages across multiple projects with packages.config, we tend to remove all existing content files as well as references with each uninstall and add new content files and references with each install of a package into each project. And all these operations are bound to be done on UI thread since they’re directly executed on dte project. We also generate few nuget events like packageinstalling, packageinstalled, packagereferenceadded, packagerefernceremoved, packageuninstalling, etc with each package installation and uninstallation. 

With jetbrains resharper installed in visual studio, this operation costs 50-60% more time since they listen to all these nuget events and do constant revaluation and build intelisense. We profiled it couple of times and saw with resharper, there is high gc cost, ~40% performance overhead for nuget addFiles/ addReference methods, resharper has their own stuff with OnAfterAddFiles, OnAfterRemoveFiles, etc, and high cpu time for context switching to main thread.
 
So we are trying to improve our update performance even with resharper tool installed. Since many of our customers use visual studio with resharper and they always face this performance overhead and complains about slow nuget update.

# Who is the customer?
Many of our customer run into this performance overhead, recently Nventive complained about updating few packages took ~6mins, given it was a simple operation of updating some 20 packages, it was huge. Like-wise almost every nuget user complains for slow update. And they all will be benefited from this.

# Evidence
We got email communication from Nventive folks about the massive UI delay during update packages and also got their generic app template to repro it in-house. After profiling, we came to know that it spends 60% time on adding/ removing files and references from projects.

# Solution
So the solution, we zeroed in is that nuget will throw batch events while executing nuget actions of installing/ uninstalling packages on per project basis which will notify resharper that there will be multiple nuget actions execution and they should hold on to their stuff until nuget is done executing all these actions. Once nuget is done executing, resharper should be able to continue with their stuff of build intellisense or manipulating references or content files.

So there will be a batch start event before nuget starts executing multiple install/ uninstall actions on a project and a batch end event after it completes executing all the actions. Existing nuget events like packageinstalling, packageinstalled, packagereferenceadded, etc... will remain unchanged and will be generated as before. **Resharper will listen for this batch start event which we fire, and just grab rest of the nuget events without executing them. And once we fire batch end event, then resharper will apply all those previous events.**
* `IVsPackageInstallerProjectEvents` - embedded interop type interface which defines BatchStart and BatchEnd event handlers which will be invoked when start batch event and end batch event is raised respectively.
* `IVsPackageProjectMetadata` - another embedded interop type interface which defines type of data passed through with these batch events.

Each batch operation will be uniquely identifiable with the batch id assigned in IVsPackageProjectMetadata with BatchStart and BatchEnd events.

# IVsPackageInstallerProjectEvents

    public interface IVsPackageInstallerProjectEvents
    {
        /// <summary>
        /// Raised before any IVsPackageInstallerEvents events are raised for a project.
        /// </summary>
        event EventHandler<IVsPackageProjectMetadata> BatchStart;

        /// <summary>
        /// Raised after all IVsPackageInstallerEvents events are raised for a project.
        /// </summary>
        event EventHandler<IVsPackageProjectMetadata> BatchEnd;
    }

# IVsPackageProjectMetadata

    public interface IVsPackageProjectMetadata
    {
        /// <summary>
        /// Unique batch id for batch start/end events of the project.
        /// </summary>
        string BatchId { get; }

        /// <summary>
        /// Name of the project.
        /// </summary>
        string ProjectName { get; }
    }

So these batch events will make sure that we only update project references or content files once per project instead of manipulating those with every package install/ uninstall.

Finally, please note that these batch events are only applicable for projects with packages.config since we never update reference or content files dynamically for project.json type projects. Project.json maintain these references through project.lock.json file.