# Upgrade API

NuGet will provide an IVS API that UWP Project system can call when a project is re targeted to a Target Platform Version >= RS2. This API will migrate the project.json based UWP project to be a PackageReference based project so it can take advantage of all the NuGet 4.0 features.

The API will look like this: 

```
    /// <summary>
    /// Contains methods to migrate a project.json based legacy project to PackageReference based project.
    /// </summary>
    public interface IVsProjectJsonToPackageReferenceMigrator
    {
        /// <summary>
        /// Migrates a UWP Project.json based project to Package Reference based project.
        /// </summary>
        /// <param name="projectUniqueName">The full path to the project that needs to be migrated</param>
        IVsProjectJsonToPackageReferenceMigrateResult MigrateProjectToPackageRef(string projectUniqueName);

    }
```

```
    /// <summary>
    /// Contains the result of the migrate operation on a legacy project.json project
    /// </summary>
    public interface IVsProjectJsonToPackageReferenceMigrateResult
    {
        /// <summary>
        /// Returns the success value of the migration operation.
        /// </summary>
        bool IsSuccess { get; }

        /// <summary>
        /// If migrate operation was unsuccessful, stores the error message in the exception.
        /// </summary>
        string ErrorMessage { get; }
    }
```

# Implementation Details

The Migration is done in three steps:

1) Migrate package dependencies to ```<PackageReference>``` in csproj.

2) Migrate runtimes in project json to ```<RuntimeIdentifiers>``` in csproj.

3) Saves a backup of the csproj and project.json file in the Backup folder in the directory where csproj file resides.

4) Saves the project.