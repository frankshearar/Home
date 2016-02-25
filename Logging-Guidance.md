This spec covers Logging in NuGet.exe and NuGet.CommandLine.Xplat as it appears in the console and output streams

1. Logging will keep a prefix of the log level: INFO, WARNING, ERROR and the like next to each method showing in the console. This allows for understanding the logs if they get piped into a file
2. Logging will write all messages to the output stream only (and never to the error stream)
3. An error summary will be produced and will be written to the error stream (so scripts can pivot on errors written as a way to determine a failure)
4. Warnings will not be written to the error stream
5. An informational summary will be written without any INFO label on it, to make it easily human readable
6. Colors will be applied in the console to Warning and Error