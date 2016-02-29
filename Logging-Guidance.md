This specification covers Logging in NuGet.exe and NuGet.CommandLine.Xplat as it appears in the console and output streams. As the dotnet CLI experience calls into NuGet.CommandLine.Xplat for some commands, this specification applies there as well.

1. Logging will keep a prefix of the log level: `info`, `warn`, `error` and the like next to each method showing in the console. This allows for understanding the logs if they get piped into a file/
1. Logging will write all messages to the output stream, STDOUT, only and never to the error stream, STDERR.
1. An error summary will be produced and will be written to the error stream so scripts can pivot on errors written as a way to determine a failure.
1. Warnings will not be written to the error stream but will be summarized to the output stream.
1. Summaries will be written without any log level prefix, to make it easily human readable.
1. Colors will be applied in the console where appropriate. For example, errors will be written in red.

Longer term, we might consider adding pivots to log messages to associate them with the project, the package, target (RID + TxM), or even the particular HTTP call made. There is a bug open to suggest a single integer can represent the stream of work, but I think we actually need 2 or 3 of them as specified above.