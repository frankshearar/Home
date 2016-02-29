## Overview

This specification covers Logging in NuGet.exe and NuGet.CommandLine.Xplat as it appears in the console and output streams. As the dotnet CLI experience calls into NuGet.CommandLine.Xplat for some commands, this specification applies there as well.

The following core requirements should be met:

1. Each line of log text will have a prefix of the log level: `info`, `warn`, `error` and the like next to each method showing in the console. This allows for understanding the logs if they get piped into a file.
1. Logging will write all messages in real time to the output stream, STDOUT, only and never to the error stream, STDERR.
1. An error summary will be produced at the end of execution and will be written to the error stream so scripts can pivot on errors written as a way to determine a failure. This is in addition to determining failure on process exit code.
1. An informational summary can be provided at the end of execution.
1. Warnings will be summarized to the output stream, before the informational summary.
1. Summaries will be written without any log level prefix, to make it easily human readable.
1. Colors will be applied in the console where appropriate. For example, errors will be written in red.

## Open Issues

1. When a multi-line log message is written, should all lines be prefixed? Or should only the first line? A common case where this happens is if an error is logged. These log messages often include multiple lines, where each line provides a different level of context via inner exception messages.
1. Should summaries have colors?
1. What are the log levels? Currently, there are the following levels, in descending order of verbosity:
   - Debug
   - Verbose
   - Information
   - Minimal
   - Warning
   - Error
   - The logging interface also includes a method to log summary lines, although "summary" is not a different log level.
1. We might consider adding pivots to log messages to associate them with the project, the package, target (RID + TxM), or even the particular HTTP call made. There is a bug open to suggest a single integer can represent the stream of work, but I think we actually need 2 or 3 of them as specified above.