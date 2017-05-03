Restore and Build generate duplicate warnings and errors in some scenarios. To improve this restore should log messages to the assets file where they can be de-duplicated with build.

Adding messages to the assets file also improves the scenario where a user builds in a different session, possibly days after doing the restore. If the restore failed build could display the original failure messages from restore to help the user debug the problem.

## Reading log messages example

```cs
var format = new LockFileFormat();
```


## Log message interface

```cs
public interface IAssetsLogMessage
{
    LogLevel Level { get; }
    NuGetLogCode Code { get; }
    string Message { get; }
    DateTimeOffset Time { get; }
    string ProjectPath { get; }
    WarningLevel WarningLevel { get; }
    string FilePath { get; }
    int StartLineNumber { get; }
    int StartColumnNumber { get; }
    int EndLineNumber { get; }
    int EndColumnNumber { get; }
    string LibraryId { get; }
    IReadOnlyList<string> TargetGraphs { get; }
}
```


## Assets file example