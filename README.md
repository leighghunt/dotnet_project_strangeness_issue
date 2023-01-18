# Unusual environment behaviour/selection with dotnet watch in parent folder

Minimal repo created for issue logged at https://github.com/dotnet/aspnetcore/issues/46144

# More info:

Inconsistent behaviour in selection of environment variable when running a project using dotnet watch from a parent directory.

This appears to have the same underlying cause or could be a duplicate of:
- https://github.com/dotnet/aspnetcore/issues/31365
- https://github.com/dotnet/sdk/issues/23183
- https://github.com/dotnet/aspnetcore/issues/44363
- https://github.com/dotnet/aspnetcore/issues/38408
- https://github.com/dotnet/aspnetcore/issues/31654

I'm recording this to give more information about these bugs and possible steps to reproduce. It seems to be caused by how `dotnet watch` is processing arguments.

### Expected Behavior

For environment selection to be consistent when run from parent folder and project folder.


### Steps To Reproduce

#### Create skeletal webapp project.
```
mkdir test
cd test
dotnet new webapp
```

#### Run from within project folder
- runs in `Development` environment, with hot reload **enabled**:
```
dotnet watch run
watch : Hot reload enabled. For a list of supported edits, see https://aka.ms/dotnet/hot-reload. Press "Ctrl + R" to restart.
watch : Building...
  Determining projects to restore...
  All projects are up-to-date for restore.
  test -> /Users/leighhunt/dev/playground/dotnetenv_test/test/bin/Debug/net6.0/test.dll
watch : Started
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: https://localhost:7108
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5041
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: /Users/leighhunt/dev/playground/dotnetenv_test/test/
```

#### Run from parent folder, with no extra arguments:
- runs in `Production` environment, with hot reload **enabled**:
```
cd ..

dotnet watch run --project test 
watch : Hot reload enabled. For a list of supported edits, see https://aka.ms/dotnet/hot-reload. Press "Ctrl + R" to restart.
watch : Building...
  Determining projects to restore...
  All projects are up-to-date for restore.
  test -> /Users/leighhunt/dev/playground/dotnetenv_test/test/bin/Debug/net6.0/test.dll
watch : Started
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5000
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: https://localhost:5001
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Production
info: Microsoft.Hosting.Lifetime[0]
      Content root path: /Users/leighhunt/dev/playground/dotnetenv_test/test/
watch : Shutdown requested. Press Ctrl+C again to force exit.
info: Microsoft.Hosting.Lifetime[0]
      Application is shutting down...
```

#### Run from parent folder, with extra arguments:
- runs in `Development` environment, with hot reload **DISABLED**:
```dotnetenv_test dotnet watch run --project test --foo bar
watch : Started
Building...
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: https://localhost:7108
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5041
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: /Users/leighhunt/dev/playground/dotnetenv_test/test/
^Cwatch : Shutdown requested. Press Ctrl+C again to force exit.
info: Microsoft.Hosting.Lifetime[0]
      Application is shutting down...
watch : Exited
```

#### Run from parent folder without run, but with extra argument:
- gets upset and misinterprets argument as `dotnet---foo` - probably caused by same issue as described in https://github.com/dotnet/aspnetcore/issues/31654#issuecomment-817051019
```
dotnet watch  --project test --foo bar 
watch : Started
Could not execute because the specified command or file was not found.
Possible reasons for this include:
  * You misspelled a built-in dotnet command.
  * You intended to execute a .NET program, but dotnet---foo does not exist.
  * You intended to run a global tool, but a dotnet-prefixed executable with this name could not be found on the PATH.
watch : Exited with error code 1
watch : Waiting for a file to change before restarting dotnet...
^Cwatch : Shutdown requested. Press Ctrl+C again to force exit.
```

Minimal repo, though not really necessary(!) - https://github.com/leighghunt/dotnet_project_strangeness_issue

### Exceptions (if any)

_No response_

### .NET Version

6.0.101

### Anything else?

```
dotnet --list-sdks    
6.0.101 [/usr/local/share/dotnet/sdk]

dotnet --info     
.NET SDK (reflecting any global.json):
 Version:   6.0.101
 Commit:    ef49f6213a

Runtime Environment:
 OS Name:     Mac OS X
 OS Version:  12.5
 OS Platform: Darwin
 RID:         osx.12-arm64
 Base Path:   /usr/local/share/dotnet/sdk/6.0.101/

Host (useful for support):
  Version: 6.0.1
  Commit:  3a25a7f1cc

.NET SDKs installed:
  6.0.101 [/usr/local/share/dotnet/sdk]

.NET runtimes installed:
  Microsoft.AspNetCore.App 6.0.1 [/usr/local/share/dotnet/shared/Microsoft.AspNetCore.App]
  Microsoft.NETCore.App 6.0.1 [/usr/local/share/dotnet/shared/Microsoft.NETCore.App]

To install additional .NET runtimes or SDKs:
  https://aka.ms/dotnet-download
```
