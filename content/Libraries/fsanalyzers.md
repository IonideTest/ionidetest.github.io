---
title: FSharp.Analyzers.SDK
menu_order: 1
---

# FSharp.Analyzers.SDK

**GitHub link:** [https://github.com/ionide/FSharp.Analyzers.SDK](https://github.com/ionide/FSharp.Analyzers.SDK)
**License:** [MIT](https://github.com/ionide/FSharp.Analyzers.SDK/blob/master/LICENSE.md)

`FSharp.Analyzers.SDK` is a library used for building custom analyzers for FSAC / F# editors. F# analyzers are live, real-time, project-based plugins that enables diagnosis of source code and surfacing of custom errors, warnings and code fixes into the editor. They're heavily influenced and inspired by [Roslyn Analyzers](https://docs.microsoft.com/en-us/visualstudio/code-quality/roslyn-analyzers-overview?view=vs-2019).

Read more about F# Analyzers here - [https://medium.com/lambda-factory/introducing-f-analyzers-772487889429](https://medium.com/lambda-factory/introducing-f-analyzers-772487889429)

## Writing Analyzers

Analyzers that are consumed by this SDK and from Ionide are simply .NET core class libraries. These class libraries expose a *value* of type `Analyzer` which is effectively a function that has input of type `Context` and returns a list of `Message` records:

```fsharp
module BadCodeAnalyzer

open FSharp.Analyzers.SDK

[<Analyzer>]
let badCodeAnalyzer : Analyzer =
  fun (context: Context) =
    // inspect context to determine the error/warning messages
    [   ]
```

Notice how we expose the function `BadCodeAnalyzer.badCodeAnalyzer` with an attribute `[<Analyzer>]` that allows the SDK to detect the function. The input `Context` is a record that contains information about a single F# file such as the typed AST, the AST, the file content, the file name and more. The SDK runs this function against all files of a project during editing. The output messages that come out of the function are eventually used by Ionide to highlight the inspected code as a warning or error depending on the `Severity` level of each message.

### Analyzer Requirements

Analyzers are .NET Core class libraries and they are distributed as such. However, since the SDK relies on dynamically loading the analyzers during runtime, there are some requirements to get them to work properly:
 - The analyzer class library has to target the `netcoreapp2.0` framework
 - The analyzer has to reference the latest `FSharp.Analyzers.SDK` (at least the version used by FsAutoComplete which is subsequently used by Ionide)

### Packaging and Distribution

Since analyzers are just .NET core libraries, you can distribute them to the nuget registry just like you would with a normal .NET package. Simply run `dotnet pack --configuration Release` against the analyzer project to get a nuget package and publish it with

```
dotnet nuget push {NugetPackageFullPath} -s nuget.org -k {NugetApiKey}
```

However, the story is different and slightly more complicated when your analyzer package has third-party dependencies also coming from nuget. Since the SDK dynamically loads the package assemblies (`.dll` files), the assemblies of the dependencies has be located *next* to the main assembly of the analyzer. Using `dotnet pack` will **not** include these dependencies into the output Nuget package. More specifically, the `./lib/netcoreapp2.0` directory of the nuget package must have all the required assemblies, also those from third-party packages. In order to package the analyzer properly with all the assemblies, you need to take the output you get from running:

```
dotnet publish --configuration Release --framework netcoreapp2.0
```

against the analyzer project and put every file from that output into the `./lib/netcoreapp2.0` directory of the nuget package. This requires some manual work by unzipping the nuget package first (because it is just an archive), modifying the directories then zipping the package again. It can be done using a FAKE build target to automate the work:

```fsharp
// make ZipFile available
#r "System.IO.Compression.FileSystem.dll"

let releaseNotes = ReleaseNotes.load "RELEASE_NOTES.md"

Target.create "PackAnalyzer" (fun _ ->
    let analyzerProject = "src" </> "BadCodeAnalyzer"
    let args =
        [
            "pack"
            "--configuration Release"
            sprintf "/p:PackageVersion=%s" releaseNotes.NugetVersion
            sprintf "/p:PackageReleaseNotes=\"%s\"" (String.concat "\n" releaseNotes.Notes)
            sprintf "--output %s" (__SOURCE_DIRECTORY__ </> "dist")
        ]

    // create initial nuget package
    let exitCode = Shell.Exec("dotnet", String.concat " " args, analyzerProject)
    if exitCode <> 0 then
        failwith "dotnet pack failed"
    else
        match Shell.Exec("dotnet", "publish --configuration Release --framework netcoreapp2.0", analyzerProject) with
        | 0 ->
            let nupkg =
                System.IO.Directory.GetFiles(__SOURCE_DIRECTORY__ </> "dist")
                |> Seq.head
                |> IO.Path.GetFullPath

            let nugetParent = DirectoryInfo(nupkg).Parent.FullName
            let nugetFileName = IO.Path.GetFileNameWithoutExtension(nupkg)

            let publishPath = analyzerProject </> "bin" </> "Release" </> "netcoreapp2.0" </> "publish"
            // Unzip the nuget
            ZipFile.ExtractToDirectory(nupkg, nugetParent </> nugetFileName)
            // delete the initial nuget package
            File.Delete nupkg
            // remove stuff from ./lib/netcoreapp2.0
            Shell.deleteDir (nugetParent </> nugetFileName </> "lib" </> "netcoreapp2.0")
            // move the output of publish folder into the ./lib/netcoreapp2.0 directory
            Shell.copyDir (nugetParent </> nugetFileName </> "lib" </> "netcoreapp2.0") publishPath (fun _ -> true)
            // re-create the nuget package
            ZipFile.CreateFromDirectory(nugetParent </> nugetFileName, nupkg)
            // delete intermediate directory
            Shell.deleteDir(nugetParent </> nugetFileName)
        | _ ->
            failwith "dotnet publish failed"
)
```
