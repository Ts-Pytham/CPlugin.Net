# CPlugin.Net

[![CPlugin.Net](https://img.shields.io/nuget/vpre/CPlugin.Net?label=CPlugin.Net%20-%20nuget&color=red)](https://www.nuget.org/packages/CPlugin.Net)
[![downloads](https://img.shields.io/nuget/dt/CPlugin.Net?color=yellow)](https://www.nuget.org/packages/CPlugin.Net)

[![CPlugin.Net.Attributes](https://img.shields.io/nuget/vpre/CPlugin.Net.Attributes?label=CPlugin.Net.Attributes%20-%20nuget&color=red)](https://www.nuget.org/packages/CPlugin.Net.Attributes)
[![downloads](https://img.shields.io/nuget/dt/CPlugin.Net.Attributes?color=yellow)](https://www.nuget.org/packages/CPlugin.Net.Attributes)

[![CPlugin.Net-logo](https://raw.githubusercontent.com/MrDave1999/CPlugin.Net/bd7e7c8787e5a1b4987cd5a506e680261dce19b0/plugin-logo.png)](https://github.com/MrDave1999/CPlugin.Net)

A simple library that allows to implement a plugin-based architecture.

See the [API documentation](https://mrdave1999.github.io/CPlugin.Net/api/CPlugin.Net.html) for more information on this project.

## Index

- [Features](#features)
- [Limitations](#limitations)
- [Why did I create this library?](#why-did-i-create-this-library)
- [What is Plug-in Architecture?](#what-is-plug-in-architecture)
  - [Where can I apply it?](#where-can-i-apply-it)
  - [Technical challenges](#technical-challenges)
- [Installation](#installation)
- [Overview](#overview)
  - [Get plugin names from .json file](#get-plugin-names-from-json-file)
  - [Get plugin names from .env file](#get-plugin-names-from-env-file)
  - [Load plugins from the host application](#load-plugins-from-the-host-application)
  - [Get the loaded assemblies](#get-the-loaded-assemblies)
  - [Communication between host application and plugins](#communication-between-host-application-and-plugins)
  - [Search for subtypes that implement the contract](#search-for-subtypes-that-implement-the-contract)
  - [Apply PluginAttribute type to plugins](#apply-pluginattribute-type-to-plugins)
  - [Creation of a Directory.Build.props file](#creation-of-a-directorybuildprops-file)
    - [Configuration](#configuration)
    - [ProjectRootDir](#projectrootdir)
    - [OutDir](#outdir)
    - [EnableDynamicLoading](#enabledynamicloading)
    - [ProjectReference](#projectreference)
    - [PackageReference](#packagereference)
  - [Copy plugins to publishing directory](#copy-plugins-to-publishing-directory)
- [Samples](#samples)
- [References](#references)
- [Contribution](#contribution)

## Features

This library contains these features:
- Loading of assemblies in different contexts to avoid dependency conflicts (see [AssemblyLoadContext](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.loader.assemblyloadcontext?view=net-8.0)).
- Resolves the dependencies of each plugin using [AssemblyDependencyResolver](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.loader.assemblydependencyresolver?view=net-8.0).
- Gets plugin names from a configuration file as `.json` or `.env`.
- Gets full paths of plugins (plugin locator).
- It is able to create the instance of the type that implements the contract (a contract previously used by the plugin).
- A logger indicating that the plugin was successfully loaded.

## Limitations

This library contains these limitations:
- The plugin loader is not thread-safe.
- There is no support for unload plugins.
- And there may be more I don't know about..

## Why did I create this library?

- I designed this library for use it in the [DentallApp](https://github.com/DentallApp/back-end) project and for other projects according to my needs.
- I have tested many frameworks related to plugins but none of them worked as it should. I had a lot of problems.
- I wanted to share my knowledge with the community. I love open source.
- I'm a big fan of plugin-based architecture. I always had the desire to create my own plugin system ever since I was playing [SA-MP](https://www.sa-mp.mp) (San Andreas Multiplayer, a multiplayer mod for GTA San Andreas).

  In SA-MP it is possible to extend the functionalities provided by the game server (sampserver) without having to know its source code, thanks to the plugins. Amazing!

## What is Plug-in Architecture?

It consists of a host application (*or a main application*) provides public API which the plug-in can use, including a way for plug-ins to load into the host application. Plug-ins depend on the services (public API) provided by the host application and do not usually work by themselves. Conversely, the host application operates independently of the plug-ins, making it possible for developers to create plug-in projects without making changes to the host application or knowing how it works.

**Rules to be complied with:**
- The host application must not be coupled to any plug-in. It must not know about their existence.
- The host application must be able to run even if no plug-in is loaded.
- Plug-ins only depend on the public API that exposes the host application.

There are three components for this pattern:
- **Host application:** Represents the main application that contains the basic functionalities and is also responsible for loading the plug-ins (only those that are necessary).
- **Contracts:** Represents a well-defined API that allows communication between the host application and the plug-ins.
- **Plug-ins:** Represents optional modules whose sole purpose is to add more functionality to the host application without having to make changes to its source code. This definition follows the [open-closed principle](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle).

### Where can I apply it?

> An example where this pattern can be applied is this:

An ERP software has many modules such as accounting, project management, payroll, treasury, sales, among others. However, what happens if the customer does not need all the modules? What if the customer only needs the accounting module? 

That's where the plugin-based architecture comes in. You can make your system flexible by converting your modules into plug-ins. This way the customer will not have access to other functionalities, only the ones he/she has purchased.

This is amazing, because if your main application has a configuration file, you can write there the plug-ins to be loaded for this customer. Oh yeah!

This also has the benefit of reducing the size of an application by not loading unused features.

### Technical challenges

When implementing this pattern in .NET there can be a number of technical challenges:

- Define the API that is shared between the host application and the plugins. The API must be stable enough; otherwise, it may lead to breaking changes, so this will affect all plugins.

- Resolve the dependencies of each plugin; failure to do so may cause errors when running the host application. In this case you should read about [AssemblyDependencyResolver](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.loader.assemblydependencyresolver?view=net-8.0).

- Several plugin can use the same dependency but with different version. Therefore, the plugins must be isolated in different contexts. In this case you should read about [AssemblyLoadContext](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.loader.assemblyloadcontext?view=net-8.0).

- Ideally, plugins should not depend on each other (reduce coupling), but in such cases a mechanism must be found that allows them to communicate with each other (e.g. a message broker).

- There are cases where the host application and plugins reference the same dependency, so in each output directory there will be a copy of that dependency. This can cause fatal errors when running the host application.

  *Example:* *Could not load type 'Example.Contracts.ICommand' from assembly 'MyPlugin1'*

To correctly implement this pattern in .NET, it is necessary to know how `AssemblyLoadContext` works. This [article](https://tsuyoshiushio.medium.com/understand-advanced-assemblyloadcontext-with-c-16a9d0cfeae3) explains it very well.

## Installation

Install the main package using dotnet CLI:
```sh
dotnet add package CPlugin.Net --prerelease
```
This package was designed to be used in host applications such as a web api or a console application.

You must also install this secondary package that will be used in your plugins:
```sh
dotnet add package CPlugin.Net.Attributes --prerelease
```
This package provides only one type: [PluginAttribute](https://mrdave1999.github.io/CPlugin.Net/api/CPlugin.Net.PluginAttribute.html) and is used only in plugins.

## Overview

Your host application must reference the [CPlugin.Net](https://www.nuget.org/packages/CPlugin.Net) package.

You must import the namespace types at the beginning of your class file:
```cs
using CPlugin.Net;
```

This library provides four main types:
- `CPluginEnvConfiguration`
- `CPluginJsonConfiguration`
- `PluginLoader`
- `TypeFinder`

See the [API documentation](https://mrdave1999.github.io/CPlugin.Net/api/CPlugin.Net.html) for more information on these types.

### Get plugin names from .json file

For this case you need to install the [Microsoft.Extensions.Configuration.Json](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json) package to read application settings from the JSON configuration file.

Your .json file must use the `Plugins` section to specify the names of each plugin.

**Example:**
```json
{
    "Plugins": [
        "MyPlugin1.dll",
        "MyPlugin2.dll",
        "MyPlugin3.dll"
    ]
}
```
Then you can use the `CPluginJsonConfiguration` type to get the plugin files.
```cs
var configurationRoot = new ConfigurationBuilder()
    .AddJsonFile("./appsettings.json")
    .Build();
var jsonConfiguration = new CPluginJsonConfiguration(configurationRoot);
List<string> pluginFiles = jsonConfiguration.GetPluginFiles().ToList();
```
`GetPluginFiles` method will get the full path to each plugin that is in the `Plugins` section of the .json file.

**Example:**
```sh
/home/admin/HostApplication/bin/Debug/net8.0/plugins/MyPlugin1/MyPlugin1.dll
/home/admin/HostApplication/bin/Debug/net8.0/plugins/MyPlugin2/MyPlugin2.dll
/home/admin/HostApplication/bin/Debug/net8.0/plugins/MyPlugin3/MyPlugin3.dll
```
It is very important that the plugins are always in the `plugins` folder and in their own directory as in the previous example.

`MyPlugin1.dll` is inside the `MyPlugin1` directory and in turn, it is in the `plugins` directory.

Why should this be so? Well, the plugin loader must somehow locate the plugins, right? 

### Get plugin names from .env file

You can also put the names of each plugin in an .env file, although for this you must install a dotenv package that allows you to read and parse .env files. Use the [DotEnv.Core](https://www.nuget.org/packages/DotEnv.Core) package.

Your .env file must use the `PLUGINS` key to specify the names of each plugin.

**Example:**
```.env
PLUGINS=MyPlugin1.dll MyPlugin2.dll MyPlugin3.dll
```
Plugin files must be separated by spaces. However, you can also use multi-line values.

**Example:**
```.env
PLUGINS="
MyPlugin1.dll
MyPlugin2.dll
MyPlugin3.dll
"
```
Then you can use the `CPluginEnvConfiguration` type to get the plugin files.
```cs
// Load the .env file.
new DotEnv.Core.EnvLoader()
    .AddEnvFile(".env")
    .Load();
var envConfiguration = new CPluginEnvConfiguration();
List<string> pluginFiles = envConfiguration.GetPluginFiles().ToList();
```
`GetPluginFiles` method will get the full path to each plugin that is in the `PLUGINS` key of the .env file.

**Example:**
```sh
/home/admin/HostApplication/bin/Debug/net8.0/plugins/MyPlugin1/MyPlugin1.dll
/home/admin/HostApplication/bin/Debug/net8.0/plugins/MyPlugin2/MyPlugin2.dll
/home/admin/HostApplication/bin/Debug/net8.0/plugins/MyPlugin3/MyPlugin3.dll
```
As mentioned in the previous section. The plugins must be in the `plugins` directory and in their own directory so that the plugin loader can locate them.

> I like this approach because the path is calculated automatically. You no longer have to hardcode the path manually.

### Load plugins from the host application

Your host application is responsible for loading the plugins at runtime using the `PluginLoader` type.
In the `Program.cs` (entry point) call the plugin loader.

**An example using as configuration source a .env file:**
```cs
new DotEnv.Core.EnvLoader()
    .AddEnvFile(".env")
    .Load();
var envConfiguration = new CPluginEnvConfiguration();
// Loads the plugins from the .env file.
PluginLoader.Load(envConfiguration);
```

**An example using as configuration source a .json file:**
```cs
var configurationRoot = new ConfigurationBuilder()
    .AddJsonFile("./appsettings.json")
    .Build();
var jsonConfiguration = new CPluginJsonConfiguration(configurationRoot);
// Loads the plugins from the .appsettings file.
PluginLoader.Load(jsonConfiguration);
```

`Load` method will load the plugin in a different context, so it doesn't matter if your plugins have conflicting dependencies. All this thanks to [AssemblyLoadContext](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.loader.assemblyloadcontext?view=net-8.0).

It is important to know that the `PluginLoader.Load` is idempotent, so if you call the method several times it will have the same effect as if it had been the first call. It will not load the same plugin in the current process in which the application is running.

### Get the loaded assemblies

Each plugin represents an [assembly](https://learn.microsoft.com/en-us/dotnet/standard/assembly) containing a collection of types and methods.

After loading plugins from a configuration source, you can obtain the loaded assemblies using the `Assemblies` property of the `PluginLoader` type.
```cs
PluginLoader.Load(configuration);
IEnumerable<Assembly> assemblies = PluginLoader.Assemblies;
```
This property is very useful when you want to add the loaded assemblies to a third-party dependency (Fluent Validation, Scrutor, MediatR, among others) so that can search the types.

**An example using ASP.NET Core as host application:**
```cs
var builder = WebApplication.CreateBuilder(args);
var jsonConfiguration = new CPluginJsonConfiguration(builder.Configuration);
PluginLoader.Load(jsonConfiguration);
IMvcBuilder mvcBuilder = builder.Services.AddControllers();
foreach (Assembly assembly in PluginLoader.Assemblies)
{
    // This allows to register the controllers for each loaded plugin.
    mvcBuilder.PartManager.ApplicationParts.Add(new AssemblyPart(assembly));
}
```
If your plugins have controllers, then you must make the plugin part of the application; otherwise, the controllers will never be recognized by the host.

### Communication between host application and plugins

The host application does not have a direct reference to the plugins, there is no such coupling, so how do they communicate?

These are communicated through **contracts**. The host application must expose a set of contracts where each plugin must implement it.

These contracts can be represented by interfaces or abstract classes of C#. Remember a contract is only a specification, it indicates the "what you do" but not the "how you do it". It is just a set of rules that each plugin must follow.

For example, we can create a project called `Plugin.Contracts` and it contains the following contract:
```cs
public interface ICommand
{
    string Name { get; }
    string Description { get; }
    int Execute();
}
```
This contract indicates three rules that the plugin must comply with:
- Specify the name of the plugin.
- A description of the plugin.
- Execute a command as an entry point.

### Search for subtypes that implement the contract

Following the example above, the `ICommand` interface represents the supertype, so it can have many subtypes, which means that there will be concrete implementations.

These subtypes will be encapsulated in their own plugins, so the host application does not know about them (has no idea about them). 

So a mechanism must be applied so that the host application can create the instance of the subtype that implements the supertype when the application is running.

Therefore, you can use the `TypeFinder` type in your host application after you have loaded the plugins.
```cs
var configurationRoot = new ConfigurationBuilder()
    .AddJsonFile("./appsettings.json")
    .Build();
var jsonConfiguration = new CPluginJsonConfiguration(configurationRoot);
PluginLoader.Load(jsonConfiguration);

IEnumerable<ICommand> commands = TypeFinder.FindSubtypesOf<ICommand>();
foreach(ICommand command in commands)
{
    command.Execute();
}
```
`FindSubtypesOf` method will search for the subtypes of `ICommand` in each plugin that has been loaded; if no subtype is found, it returns an empty enumerable.

For this method to work correctly, each plugin must use the [PluginAttribute](https://mrdave1999.github.io/CPlugin.Net/api/CPlugin.Net.PluginAttribute.html) type to specify the subtypes. This is mandatory because the `TypeFinder` type creates the instances of the subtypes using this attribute.

### Apply PluginAttribute type to plugins

This attribute must be applied at the assembly level in the plugin project. Do not forget to install the [CPlugin.Net.Attributes](https://www.nuget.org/packages/CPlugin.Net.Attributes) package in the plugin project in order to be able to use the [PluginAttribute](https://mrdave1999.github.io/CPlugin.Net/api/CPlugin.Net.PluginAttribute.html) type.

**Example:**
```cs
using CPlugin.Net;
using Project.PluginExample;
using Plugin.Contracts;

// This line is mandatory.
[assembly: Plugin(typeof(HelloWorldCommand))]

namespace Project.PluginExample;

public class HelloWorldCommand : ICommand
{
    public string Name => nameof(HelloWorldCommand);
    public string Description => "Outputs Hello Word";

    public int Execute()
    {
        System.Console.WriteLine("Hello World!");
        return 0;
    }
}
```
`HelloWorldCommand` type is a subtype of `ICommand`. Absolutely nobody knows about this implementation, not even the host application. However, this attribute is not necessary if the plugin does not need to implement any contract.

### Creation of a Directory.Build.props file

It is recommended to create a .props file that is used globally for all plugin projects. 

When you compile your plugins, the result of the compilation should be copied to the output directory of the host application, so you can specify it in the `Directory.Build.props` file.

**Given the following project structure:**
```sh
└── MyApp/
    ├── src/
    │   ├── HostApplication/
    │   │   ├── bin/Debug/net8.0/plugins/
    │   │   │   ├── MyPlugin1/
    │   │   │   │   └── MyPlugin1.dll
    │   │   │   ├── MyPlugin2/
    │   │   │   │   └── MyPlugin2.dll
    │   │   │   └── MyPlugin3/
    │   │   │       └── MyPlugin3.dll
    │   │   ├── Program.cs
    │   │   └── HostApplication.csproj
    │   ├── Plugins/
    │   │   ├── MyPlugin1/
    │   │   │   └── MyPlugin1.csproj
    │   │   ├── MyPlugin2/
    │   │   │   └── MyPlugin2.csproj
    │   │   ├── MyPlugin3/
    │   │   │   └── MyPlugin3.csproj
    │   │   └── Directory.Build.props
    │   └── Contracts/
    │       ├── ICommand.cs
    │       ├── IWebStartup.cs
    │       └── Contracts.csproj
    ├── tests/
    └── MyApp.sln
```
*NOTE: This structure is only an example.*

`Plugins` directory must contain the `Directory.Build.props` file that will be shared for each plugin project.

The .props file could look like this:
```xml
<Project>
  <PropertyGroup>
    <Configuration Condition="$(Configuration) == ''">Debug</Configuration>
    <ProjectRootDir>$([MSBuild]::GetDirectoryNameOfFileAbove($(MSBuildThisFileDirectory), 'MyApp.sln'))</ProjectRootDir>
    <OutDir>$(ProjectRootDir)/src/HostApplication/bin/$(Configuration)/$(TargetFramework)/plugins/$(MSBuildProjectName)</OutDir>
    <EnableDynamicLoading>true</EnableDynamicLoading>
  </PropertyGroup>
  
  <ItemGroup>
    <ProjectReference Include="$(ProjectRootDir)/src/Contracts/Contracts.csproj">
      <Private>false</Private>
      <ExcludeAssets>runtime</ExcludeAssets>
    </ProjectReference>

    <PackageReference Include="CPlugin.Net.Attributes" Version="1.0.0">
      <ExcludeAssets>runtime</ExcludeAssets>
    </PackageReference>
  </ItemGroup>
</Project>
```

### Explanation

#### Configuration
```xml
<Configuration Condition="$(Configuration) == ''">Debug</Configuration>
```
If the `$(Configuration)` property is not defined, then by default it will be **Debug**. This is useful when you want to compile a particular plugin project with **dotnet build**, to ensure that the configuration is set even if its value is an empty string.

#### ProjectRootDir
```xml
<ProjectRootDir>$([MSBuild]::GetDirectoryNameOfFileAbove($(MSBuildThisFileDirectory), 'MyApp.sln'))</ProjectRootDir>
```
[GetDirectoryNameOfFileAbove is MSBuild property function](https://learn.microsoft.com/en-us/visualstudio/msbuild/property-functions?view=vs-2022#msbuild-property-functions) that allows you to search for a specific file in the parent directories.

*Function signature:*
```cs
GetDirectoryNameOfFileAbove(string startingDirectory, string fileName)
```
*Description:*
> Locate and return the directory of a file in either the directory specified or a location in the directory structure above that directory.

In this case this function is used to obtain the root directory of the project, so the idea is to search the `MyApp.sln` file until it is found.

`$(MSBuildThisFileDirectory)` will give us the path where the current Directory.Build.props file is located (in this case it is `MyApp/src/Plugins/`).

#### OutDir
```xml
<OutDir>$(ProjectRootDir)/src/HostApplication/bin/$(Configuration)/$(TargetFramework)/plugins/$(MSBuildProjectName)</OutDir>
```
Once the root directory of the project is obtained, it is necessary to indicate where the compilation result of our plugins should be copied.

`OutDir` will contain these possible values:
```sh
/home/admin/MyApp/src/HostApplication/bin/Debug/net8.0/plugins/MyPlugin1/
/home/admin/MyApp/src/HostApplication/bin/Debug/net8.0/plugins/MyPlugin2/
/home/admin/MyApp/src/HostApplication/bin/Debug/net8.0/plugins/MyPlugin3/
```
Remember that the host application needs to know where to locate the plugins in order to load it.

#### EnableDynamicLoading
```xml
<EnableDynamicLoading>true</EnableDynamicLoading>
```
> NuGet references are copied locally.

See [EnableDynamicLoading](https://learn.microsoft.com/en-us/dotnet/core/project-sdk/msbuild-props#enabledynamicloading).

This tag is necessary because the third-party dependencies used by the plugin must be copied to the output directory; otherwise, the host application may throw an exception when loading the plugins, because the NuGet references are not found.

#### ProjectReference
```xml
<ProjectReference Include="$(ProjectRootDir)/src/Contracts/Contracts.csproj">
    <Private>false</Private>
    <ExcludeAssets>runtime</ExcludeAssets>
</ProjectReference>
```
These are the contracts shared between the host application and the plugins and in order not to have to copy this same reference in each .csproj file of a plugin, we can add it at once in the `Directory.Build.props` file.

`<Private>false</Private>`. This tells MSBuild not to copy **Contracts.dll** to the plugin output directory.

`<ExcludeAssets>runtime</ExcludeAssets>`. This setting has the same effect as `<Private>false</Private>` but works on package references 
that the Contracts project or one of its dependencies may include.

These tags are necessary because it is not recommended to share assemblies between the host application and the plugins. For example, the assembly like Contracts.dll is used by both the host application and plugins, if you share it, you may get the following confusing error when running the host application:
> Unable to cast object of type 'Contracts.ICommand' to type 'Contracts.ICommand'.

Therefore the assembly as Contracts.dll should only be copied to the output directory of the host application so that it is loaded only in the default context.

#### PackageReference
```xml
<PackageReference Include="CPlugin.Net.Attributes" Version="1.0.0">
    <ExcludeAssets>runtime</ExcludeAssets>
</PackageReference>
```
`<ExcludeAssets>runtime</ExcludeAssets>`. This avoids having to copy `CPlugin.Net.Attributes.dll` and its dependencies to the plugin output directory.

All plugins must reference the CPlugin.Net.Attributes package, however, you should not copy the `CPlugin.Net.Attributes.dll` assembly to the output directory of the plugin project. This is because the host application already contains such an assembly.

### Copy plugins to publishing directory

You can copy the plugins to the publishing directory of the host application.

Add this in the host application .csproj file:
```xml
<ItemGroup>
  <!-- 
    Copy the plugins directory to the publish directory.
    This copies the directories and subdirectories (including files with extension) 
    from the plugins folder to the publish directory.
    For this to work, the plug-ins must be compiled.
  -->
  <Content 
    Include="bin\$(Configuration)\$(TargetFramework)\plugins\**" 
    CopyToPublishDirectory="PreserveNewest"
    TargetPath="plugins\%(RecursiveDir)\%(Filename)%(Extension)"
  />
</ItemGroup>
```
Do not forget to compile the plugins before publishing. The easiest way is to have a solution file (.sln) with all the plugins so you can compile it at once.

## Samples

You can find a complete and functional example in these projects:

- [Example.HostConsoleApp](https://github.com/MrDave1999/CPlugin.Net/tree/master/samples/HostApplications/ConsoleApp)
- [Example.HostWebApi](https://github.com/MrDave1999/CPlugin.Net/tree/master/samples/HostApplications/WebApi)
- [Example.Plugins](https://github.com/MrDave1999/CPlugin.Net/tree/master/samples/Plugins)
- [DentallApp.BackEnd.Host](https://github.com/DentallApp/back-end/blob/dev/src/HostApplication/Startup.cs)
- [DentallApp.BackEnd.Plugins](https://github.com/DentallApp/back-end/tree/dev/src/Plugins)

## References

- [Create a .NET Core application with plugins](https://learn.microsoft.com/en-us/dotnet/core/tutorials/creating-app-with-plugin-support)
- [A sample plugin model for ASP.NET Core applications](https://github.com/davidfowl/WebApplicationPlugins#webapplicationplugins)
- [Understand Advanced AssemblyLoadContext with C#](https://tsuyoshiushio.medium.com/understand-advanced-assemblyloadcontext-with-c-16a9d0cfeae3)
- [Plug-in Architecture](https://medium.com/omarelgabrys-blog/plug-in-architecture-dec207291800)
- [Plug-in (computing)](https://en.wikipedia.org/wiki/Plug-in_(computing))

## Contribution

Any contribution is welcome! Remember that you can contribute not only in the code, but also in the documentation or even improve the tests.

Follow the steps below:

- Fork it
- Create your feature branch (git checkout -b my-new-change)
- Commit your changes (git commit -am 'Add some change')
- Push to the branch (git push origin my-new-change)
- Create new Pull Request