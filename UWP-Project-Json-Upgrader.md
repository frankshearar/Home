# Upgrade API

NuGet will provide an IVS API that UWP Project system can call when a project is re targeted to a Minimum Platform Version >= RS2. This API will migrate the project.json based UWP project to be a PackageReference based project so it can take advantage of all the NuGet 4.0 features.

The API will look like this: 

```
    /// <summary>
    /// Contains methods to migrate a UWP project to PackageReference based project.
    /// </summary>
    public interface IVsProjectJsonMigrator
    {
        /// <summary>
        /// Migrateds a UWP Project.json based project to Package Reference based project.
        /// </summary>
        /// <param name="project">The DTE project that needs to be migrated</param>
        Task<IVsProjectJsonMigrateResult> MigrateProjectToPackageRef(Project project);
        
    }
```

```
    /// <summary>
    /// Contains the result of the migrate operation on a UWP project
    /// </summary>
    public interface IVsProjectJsonMigrateResult
    {
        /// <summary>
        /// Returns the success value of the migration operation.
        /// </summary>
        bool IsSuccess { get; }

        /// <summary>
        /// If migrate operation was successful, stores the path to the backup project file on disk.
        /// </summary>
        string BackupProjectFile { get; }

        /// <summary>
        /// If migrate operation was successful, stores the path to the backup project.json file on disk.
        /// </summary>
        string BackupProjectJsonFile { get; }
    }
```

# Implementation Details

The Migration is done in three steps:

1) Migrate package dependencies to Package References in csproj.
2) Migrate runtimes in project json to ```<RuntimeIdentifiers>``` in csproj.
3) Migrate root properties in project json like ****Description, Copyright, Version, Authors, Language****